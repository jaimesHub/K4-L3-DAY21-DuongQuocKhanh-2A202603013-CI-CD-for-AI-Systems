# Bước 2 - Hướng Dẫn AWS (S3 + EC2)

Tài liệu này hướng dẫn từng bước để thiết lập pipeline CI/CD cho AWS, thay thế cho GCP trong tài liệu `tasks/buoc-2.md`.

---

## Chuẩn Bị Biến Môi Trường

Để dễ dàng sao chép lệnh, hãy đặt các biến này tại đầu terminal:

```bash
export AWS_ACCOUNT_ID="123456789012"        # Thay bằng Account ID của bạn
export AWS_REGION="us-east-1"               # Hoặc region gần bạn nhất
export BUCKET_NAME="income-lab-${AWS_ACCOUNT_ID}"  # Tên bucket phải duy nhất
export VM_KEY_NAME="income-lab-key"         # Tên key pair EC2
export VM_SECURITY_GROUP="income-api-sg"
export VM_INSTANCE_NAME="income-api"
```

---

## Bước 1: Tạo S3 Bucket

Tạo bucket và cấu hình chặn truy cập công khai:

```bash
# Tạo bucket
aws s3 mb s3://$BUCKET_NAME --region $AWS_REGION

# Chặn tất cả truy cập công khai
aws s3api put-public-access-block \
  --bucket $BUCKET_NAME \
  --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Xác nhận bucket tồn tại
aws s3 ls | grep $BUCKET_NAME
```

---

## Bước 2: Tạo IAM User Cho Lab

Tạo một IAM user với quyền tối thiểu chỉ trên bucket của bạn:

### 2.1: Tạo IAM User

```bash
aws iam create-user --user-name income-lab-user

# Xác nhận user được tạo
aws iam get-user --user-name income-lab-user
```

### 2.2: Tạo IAM Policy Tối Thiểu

Tạo file `s3-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetBucketVersioning"
      ],
      "Resource": "arn:aws:s3:::BUCKET_NAME"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::BUCKET_NAME/*"
    }
  ]
}
```

Thay `BUCKET_NAME` bằng tên bucket thực tế:

```bash
# Thay $BUCKET_NAME trong policy
sed -i.bak "s/BUCKET_NAME/$BUCKET_NAME/g" s3-policy.json

# Tạo inline policy
aws iam put-user-policy \
  --user-name income-lab-user \
  --policy-name s3-income-lab-access \
  --policy-document file://s3-policy.json

# Xác nhận
aws iam get-user-policy --user-name income-lab-user --policy-name s3-income-lab-access
```

### 2.3: Tạo Access Key

```bash
aws iam create-access-key --user-name income-lab-user > /tmp/access-key.json

# Xem access key (giữ bí mật!)
cat /tmp/access-key.json
```

Lưu `AccessKeyId` và `SecretAccessKey` vào file an toàn (ví dụ: `~/.aws/credentials`) hoặc sử dụng trực tiếp trong GitHub Secrets.

**Lưu ý quan trọng:** Cặp key này sẽ được sử dụng lại ở Bước 9.1 để set GitHub Secret. Mỗi IAM user tối đa có 2 access key, nên không nên tạo key lần hai nếu vẫn còn key này.

---

## Bước 3: Cấu Hình DVC Với S3

Trên máy tính cá nhân:

```bash
# Khởi tạo DVC nếu chưa có
dvc init

# Thêm S3 remote
dvc remote add -d labstore s3://$BUCKET_NAME/dvc

# Cấu hình region (thay us-east-1 nếu cần)
dvc remote modify labstore region $AWS_REGION

# Cấu hình credentials (DVC sẽ đọc từ ~/.aws hoặc biến môi trường)
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"

# Kiểm tra cấu hình
cat .dvc/config
```

### 3.1: Theo Dõi Dữ Liệu Với DVC

```bash
# Thêm file CSV vào DVC
dvc add data/train_batch1.csv
dvc add data/holdout.csv
dvc add data/train_batch2.csv

# Commit DVC files vào git
git add data/train_batch1.csv.dvc data/holdout.csv.dvc data/train_batch2.csv.dvc \
        .gitignore .dvc/config
git commit -m "feat: track datasets with DVC for S3"

# Đẩy dữ liệu lên S3
dvc push
```

Xác nhận trên AWS S3 Console:
- Bucket `$BUCKET_NAME`
- Prefix `dvc/files/md5/` chứa các file CSV

---

## Bước 4: Tạo EC2 Instance

### 4.1: Tạo Key Pair

```bash
# Tạo key pair mới (lưu ý bạn chỉ có một cơ hội tải về)
aws ec2 create-key-pair --key-name $VM_KEY_NAME --region $AWS_REGION \
  --query 'KeyMaterial' --output text > ~/.ssh/$VM_KEY_NAME.pem

chmod 400 ~/.ssh/$VM_KEY_NAME.pem
```

### 4.2: Tạo Security Group

```bash
# Tạo security group
aws ec2 create-security-group \
  --group-name $VM_SECURITY_GROUP \
  --description "Security group for income-api service" \
  --region $AWS_REGION

# Lấy security group ID
SG_ID=$(aws ec2 describe-security-groups \
  --filters Name=group-name,Values=$VM_SECURITY_GROUP \
  --region $AWS_REGION --query 'SecurityGroups[0].GroupId' --output text)

# Mở cổng 22 (SSH) từ bất kỳ đâu (chỉ cho lab)
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0 \
  --region $AWS_REGION

# Mở cổng 8080 (API) từ bất kỳ đâu
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID --protocol tcp --port 8080 --cidr 0.0.0.0/0 \
  --region $AWS_REGION
```

### 4.3: Khởi Tạo EC2 Instance

```bash
# Tìm AMI ID cho Ubuntu 22.04 LTS (x86)
AMI_ID=$(aws ec2 describe-images \
  --filters Name=name,Values="ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --owners 099720109477 \
  --region $AWS_REGION \
  --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
  --output text)

# Tạo instance
aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --key-name $VM_KEY_NAME \
  --security-groups $VM_SECURITY_GROUP \
  --region $AWS_REGION \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$VM_INSTANCE_NAME}]" \
  --query 'Instances[0].InstanceId' --output text > /tmp/instance-id.txt

INSTANCE_ID=$(cat /tmp/instance-id.txt)

# Chờ instance khởi động (2-3 phút)
echo "Waiting for instance to be running..."
aws ec2 wait instance-running --instance-ids $INSTANCE_ID --region $AWS_REGION

# Lấy public IP (có thể mất vài giây sau khi instance running)
sleep 10
PUBLIC_IP=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --region $AWS_REGION \
  --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "Instance IP: $PUBLIC_IP"
echo "Lưu IP này để dùng cho GitHub Secrets"
```

---

## Bước 5: Cấu Hình EC2 Instance

SSH vào instance và cài đặt dependencies:

```bash
# SSH vào VM
ssh -i ~/.ssh/$VM_KEY_NAME.pem ubuntu@$PUBLIC_IP

# Trong VM - cập nhật và cài Python
sudo apt-get update
sudo apt-get install -y python3-pip python3-venv

# Cài các thư viện cần thiết với phiên bản ghim (phải khớp với requirements.txt)
pip3 install --user "scikit-learn==1.4.2" "numpy<2" "joblib==1.4.2" "fastapi==0.111.0" "uvicorn==0.29.0" "boto3==1.34.49"

# ⚠️ LƯU Ý VỀ PHIÊN BẢN:
# VM phải cài cùng phiên bản như phiên bản train trong CI (requirements.txt).
# Nếu VM cài scikit-learn mới hơn (ví dụ 1.5+), joblib.load() sẽ báo lỗi:
#   AttributeError: '__pyx_unpickle_CyHalfBinomialLoss'
# 
# Triệu chứng: /score endpoint 500 error, log có lỗi unpickle.
# Cách khắc phục: Chạy lại lệnh pip3 trên để cài đúng phiên bản, sau đó:
#   sudo systemctl restart income-api

# Tạo thư mục cho model và code
mkdir -p ~/models ~/src

# Thoát khỏi VM
exit
```

---

## Bước 6: Sao Chép serve.py Lên VM

Từ máy tính cá nhân:

```bash
scp -i ~/.ssh/$VM_KEY_NAME.pem \
  src/serve.py \
  ubuntu@$PUBLIC_IP:~/src/serve.py
```

---

## Bước 7: Cấu Hình Systemd Service Trên VM

SSH vào VM lần nữa:

```bash
ssh -i ~/.ssh/$VM_KEY_NAME.pem ubuntu@$PUBLIC_IP
```

Tạo systemd service file:

```bash
# Lưu BUCKET_NAME vào biến
BUCKET_NAME="income-lab-YOUR_AWS_ACCOUNT_ID"
AWS_REGION="us-east-1"

# Tạo service file
sudo tee /etc/systemd/system/income-api.service > /dev/null <<EOF
[Unit]
Description=Income Model Inference Server
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu
Environment="ARTIFACT_BUCKET=$BUCKET_NAME"
Environment="AWS_DEFAULT_REGION=$AWS_REGION"
# Để sử dụng IAM role instance profile, bỏ comment dòng trên
# Để sử dụng access key, thêm:
# Environment="AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE"
# Environment="AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
ExecStart=/usr/bin/python3 /home/ubuntu/src/serve.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd
sudo systemctl daemon-reload

# Enable service (tự động khởi động khi VM restart)
sudo systemctl enable income-api

# Chưa start service (chờ pipeline đầu tiên upload model)
echo "Service đã được enable. Sẽ start sau khi pipeline upload model."

# Thoát khỏi VM
exit
```

---

## Bước 7.1: Cấu Hình IAM Instance Profile Cho VM (Khuyến Nghị Bảo Mật)

Thay vì dùng access key trong service file, sử dụng IAM instance profile (role được gắn trực tiếp vào EC2):

### 7.1.1: Tạo IAM Role Và Policy

Trên máy cá nhân (cần credentials admin):

```bash
# Tạo trust policy cho EC2
cat > /tmp/ec2-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Tạo IAM role
aws iam create-role \
  --role-name income-api-role \
  --assume-role-policy-document file:///tmp/ec2-trust-policy.json

# Tạo inline policy cho S3 artifacts (chỉ-đọc)
cat > /tmp/s3-artifacts-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::BUCKET_NAME/artifacts/*"
    }
  ]
}
EOF

# Thay BUCKET_NAME
sed -i.bak "s/BUCKET_NAME/$BUCKET_NAME/g" /tmp/s3-artifacts-policy.json

# Gắn policy vào role
aws iam put-role-policy \
  --role-name income-api-role \
  --policy-name s3-artifacts-read \
  --policy-document file:///tmp/s3-artifacts-policy.json

# Tạo instance profile
aws iam create-instance-profile --instance-profile-name income-api-profile

# Gắn role vào instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name income-api-profile \
  --role-name income-api-role

# Gắn instance profile vào EC2 instance (nếu instance đã chạy)
aws ec2 associate-iam-instance-profile \
  --iam-instance-profile Name=income-api-profile \
  --instance-id $INSTANCE_ID \
  --region $AWS_REGION

# Xóa file tạm (KHÔNG để trong repo)
rm -f /tmp/ec2-trust-policy.json /tmp/s3-artifacts-policy.json /tmp/s3-artifacts-policy.json.bak
```

### 7.1.2: Cập Nhật Systemd Service

Bỏ comment dòng `IAM role instance profile` trong service file (Bước 7, dòng 309) và comment lại các dòng `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY`:

```bash
ssh -i ~/.ssh/$VM_KEY_NAME.pem ubuntu@$PUBLIC_IP << 'EOF'
sudo tee /etc/systemd/system/income-api.service > /dev/null <<SERVICEEOF
[Unit]
Description=Income Model Inference Server
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu
Environment="ARTIFACT_BUCKET=$BUCKET_NAME"
Environment="AWS_DEFAULT_REGION=$AWS_REGION"
# Để sử dụng IAM role instance profile (khuyến nghị)
# ExecStart sẽ tự động lấy credentials từ EC2 metadata
ExecStart=/usr/bin/python3 /home/ubuntu/src/serve.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
SERVICEEOF

sudo systemctl daemon-reload
EOF
```

---

## Bước 8: Tạo SSH Key Cho GitHub Actions

Tạo key pair riêng cho GitHub Actions:

```bash
# Tạo key pair mới
ssh-keygen -t ed25519 -f ~/.ssh/income_deploy -N "" -C "github-actions-deploy"

# Lấy public key
cat ~/.ssh/income_deploy.pub
```

Thêm public key vào authorized_keys của VM:

```bash
ssh -i ~/.ssh/$VM_KEY_NAME.pem ubuntu@$PUBLIC_IP << 'EOF'
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "PUBLIC_KEY_CONTENT_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
EOF
```

Thay `PUBLIC_KEY_CONTENT_HERE` bằng nội dung của `~/.ssh/income_deploy.pub`.

---

## Bước 9: Thêm GitHub Secrets

Trên GitHub repo của bạn, vào **Settings > Secrets and variables > Actions** và thêm 5 secrets:

### 9.1: STORAGE_CREDENTIALS

Nội dung JSON:

```json
{
  "aws_access_key_id": "AKIAIOSFODNN7EXAMPLE",
  "aws_secret_access_key": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
}
```

**Cách A (Khuyến nghị nếu còn file key từ Bước 2.3):**

Nếu file `/tmp/access-key.json` từ Bước 2.3 vẫn còn, sử dụng đúng cặp key này:

```bash
python3 -c "import json; k=json.load(open('/tmp/access-key.json'))['AccessKey']; print(json.dumps({'aws_access_key_id':k['AccessKeyId'],'aws_secret_access_key':k['SecretAccessKey']}))" | gh secret set STORAGE_CREDENTIALS

# Sau khi set xong, xóa file (chỉ xóa khi đã confirm secret được set và cấu hình cục bộ xong)
rm -f /tmp/access-key.json
```

**Cách B (Nếu mất file key hoặc cần key mới):**

Trước hết kiểm tra xem đã có 2 key:

```bash
aws iam list-access-keys --user-name income-lab-user
```

Nếu đã có 2 key, xóa key không dùng:

```bash
aws iam delete-access-key --user-name income-lab-user --access-key-id AKIAXXXXXXXXXXXXX
```

Sau đó tạo key mới và set trực tiếp (trong một lệnh, không lưu lên disk):

```bash
aws iam create-access-key --user-name income-lab-user --output json | python3 -c "import json,sys; k=json.load(sys.stdin)['AccessKey']; print(json.dumps({'aws_access_key_id':k['AccessKeyId'],'aws_secret_access_key':k['SecretAccessKey']}))" | gh secret set STORAGE_CREDENTIALS
```

**Lưu ý:** 
- Nếu xóa key cục bộ (~/.aws/credentials), phải cấu hình lại: `aws configure --profile income-lab`
- Đừng tạo key thứ 2 nếu vẫn có key từ Bước 2.3; mỗi user tối đa 2 key

### 9.2: ARTIFACT_BUCKET

```bash
gh secret set ARTIFACT_BUCKET --body "$BUCKET_NAME"
```

### 9.3: SERVER_HOST

IP công khai của EC2:

```bash
gh secret set SERVER_HOST --body "$PUBLIC_IP"
```

### 9.4: SERVER_USER

```bash
gh secret set SERVER_USER --body "ubuntu"
```

### 9.5: SERVER_SSH_KEY

Private key của GitHub Actions (không có passphrase):

```bash
gh secret set SERVER_SSH_KEY < ~/.ssh/income_deploy
```

---

## Bước 10: Chạy Pipeline CI/CD Lần Đầu

Commit và push code:

```bash
# Kiểm tra status
git status

# Nếu chưa add
git add .

# Commit
git commit -m "feat: add AWS-based CI/CD pipeline and serving infrastructure"

# Push lên main
git push origin main
```

Theo dõi pipeline:

```bash
# Mở browser vào tab Actions của repo GitHub
# Hoặc dùng GitHub CLI
gh run watch
```

Chờ 4 jobs hoàn thành:
1. Unit Test
2. Train
3. Quality Gate
4. Release

---

## Bước 11: Kiểm Tra Deployment

Sau khi pipeline xanh hết, start service trên VM:

```bash
ssh -i ~/.ssh/$VM_KEY_NAME.pem ubuntu@$PUBLIC_IP \
  "sudo systemctl start income-api"

# Kiểm tra service đang chạy
ssh -i ~/.ssh/$VM_KEY_NAME.pem ubuntu@$PUBLIC_IP \
  "sudo systemctl status income-api"
```

Test API:

```bash
# Health check
curl http://$PUBLIC_IP:8080/healthz

# Dự đoán (10 đặc trưng)
curl -X POST http://$PUBLIC_IP:8080/score \
  -H "Content-Type: application/json" \
  -d '{"features": [60, 2, 5, 2, 4, 0, 1, 0, 0, 45]}'

# Mẫu khác (độ tuổi thấp hơn, học vấn cao hơn)
curl -X POST http://$PUBLIC_IP:8080/score \
  -H "Content-Type: application/json" \
  -d '{"features": [28, 2, 14, 2, 11, 0, 1, 0, 0, 45]}'
```

---

## Bước 12: Chụp Ảnh Để Nộp Bài

### Ảnh 02: GitHub Actions Jobs

- Tab Actions > Commit gần nhất
- Chụp màn hình cho thấy cả 4 jobs (unit-test, train, quality-gate, release) với trạng thái xanh (Success)

### Ảnh 04: API Endpoints

- Mở terminal, chạy 2 lệnh curl ở trên
- Chụp màn hình hiển thị:
  - IP của VM rõ ràng
  - Kết quả từ `/healthz`
  - Kết quả từ `/score` (ít nhất 1 mẫu)

### Ảnh 05: S3 Console

- AWS Console > S3
- Bucket `$BUCKET_NAME`
- Chụp cho thấy:
  - Folder `dvc/` chứa dữ liệu
  - Folder `artifacts/current/` chứa `model.joblib`

---

## Gỡ Lỗi

### Pipeline `dvc pull` thất bại

Kiểm tra:
1. `STORAGE_CREDENTIALS` secret được set đúng JSON format
2. IAM user có quyền `s3:GetObject` trên bucket
3. `.dvc/config` có remote URL đúng:

```bash
cat .dvc/config
```

### Lỗi thường gặp

**Pipeline báo lỗi `dvc pull`: 403 HeadObject**

Nguyên nhân phổ biến: `aws_access_key_id` và `aws_secret_access_key` không cùng một cặp key (AccessKeyId từ lần tạo mới, SecretAccessKey từ key cũ).

Kiểm tra số lượng key hiện tại:

```bash
aws iam list-access-keys --user-name income-lab-user
```

Nếu có 2 key và chắc chắn key nào là key cũ, xóa key không dùng:

```bash
aws iam delete-access-key --user-name income-lab-user --access-key-id AKIAXXXXXXXXXXXXX
```

Sau đó sử dụng **Cách B** từ Bước 9.1 để tạo key mới và set secret lại.

### Service không khởi động trên VM

Xem log:

```bash
ssh -i ~/.ssh/$VM_KEY_NAME.pem ubuntu@$PUBLIC_IP \
  "sudo journalctl -u income-api -n 50"
```

Nguyên nhân phổ biến:
- `ARTIFACT_BUCKET` sai trong service file
- Model chưa được upload lên S3 (chờ pipeline xanh)
- Thiếu quyền S3 từ VM

### Curl API không kết nối

- Kiểm tra security group mở cổng 8080
- Kiểm tra service running: `systemctl status income-api`
- Kiểm tra firewall trên VM: `sudo ufw allow 8080`

---

## Dọn Dẹp Tài Nguyên (Tránh Phát Sinh Chi Phí)

Khi hoàn thành lab, hãy xóa các tài nguyên AWS:

```bash
**Trước khi chạy:** dùng credentials **admin** (`aws sts get-caller-identity` phải ra user admin, không phải `income-lab-user`). Mở terminal mới thì các biến `export` bị mất, nên đặt lại ngay đầu (nếu để trống, lệnh sẽ báo `argument --region: expected one argument`). Việc dọn dẹp **không thể khôi phục**: API ngừng hoạt động, dữ liệu DVC và model trên S3 bị xóa. Hãy làm sau khi đã chụp đủ ảnh và hoàn thành mọi việc cần hạ tầng (kể cả quality gate/bonus nếu có làm).

```bash
export AWS_REGION="us-east-1"
export BUCKET_NAME="income-lab-<AWS_ACCOUNT_ID>"
# Nếu mất $INSTANCE_ID, lấy lại:
export INSTANCE_ID=$(aws ec2 describe-instances --region $AWS_REGION \
  --filters Name=tag:Name,Values=income-api Name=instance-state-name,Values=running,stopped \
  --query 'Reservations[0].Instances[0].InstanceId' --output text)

# 1. Xóa EC2 instance và chờ terminate hoàn toàn
aws ec2 terminate-instances --instance-ids $INSTANCE_ID --region $AWS_REGION
aws ec2 wait instance-terminated --instance-ids $INSTANCE_ID --region $AWS_REGION

# 2. Xóa security group và key pair (security group chỉ xóa được sau khi instance đã terminate)
aws ec2 delete-security-group --group-name income-api-sg --region $AWS_REGION
aws ec2 delete-key-pair --key-name income-lab-key --region $AWS_REGION

# 3. Xóa IAM role và instance profile của EC2 (Bước 7.1).
#    Không cần disassociate: association tự biến mất khi instance terminate.
aws iam remove-role-from-instance-profile \
  --instance-profile-name income-api-profile \
  --role-name income-api-role
aws iam delete-instance-profile --instance-profile-name income-api-profile
# Lưu ý: Tên policy thực tế có thể khác (ví dụ: read-artifacts thay vì s3-artifacts-read)
# Kiểm tra tên bằng: aws iam list-role-policies --role-name income-api-role
aws iam delete-role-policy --role-name income-api-role --policy-name s3-artifacts-read
aws iam delete-role --role-name income-api-role

# 4. Xóa S3 bucket (phải trống trước; KHÔNG khôi phục được)
aws s3 rm s3://$BUCKET_NAME --recursive
aws s3 rb s3://$BUCKET_NAME

# 5. Xóa IAM user của lab: access key, inline policy, rồi user
for k in $(aws iam list-access-keys --user-name income-lab-user --query 'AccessKeyMetadata[].AccessKeyId' --output text); do
  aws iam delete-access-key --user-name income-lab-user --access-key-id $k
done
aws iam delete-user-policy --user-name income-lab-user --policy-name s3-income-lab-access
aws iam delete-user --user-name income-lab-user

# 6. (Tùy chọn) Dọn máy local và GitHub Secrets
rm -f ~/.ssh/income-lab-key.pem ~/.ssh/income_deploy ~/.ssh/income_deploy.pub
# gh secret delete STORAGE_CREDENTIALS; gh secret delete ARTIFACT_BUCKET; gh secret delete SERVER_HOST
# gh secret delete SERVER_USER; gh secret delete SERVER_SSH_KEY
```

Sau khi dọn xong, kiểm tra không còn tài nguyên chạy: `aws ec2 describe-instances --region $AWS_REGION --query 'Reservations[].Instances[].[InstanceId,State.Name]'` và `aws s3 ls`.

---

## Thứ Tự An Toàn

1. **Luôn kiểm tra trước khi commit:**
   ```bash
   git status
   ```
   Đảm bảo không có `.aws/`, `sa-key.json`, hoặc file chứa credentials

2. **Không commit credentials và file policy tạm:**
   ```bash
   # Kiểm tra .gitignore
   cat .gitignore
   ```
   
   **Thêm vào .gitignore nếu chưa có:**
   ```
   s3-policy.json
   s3-policy.json.bak
   .aws/
   ```
   
   **Xóa file tạm:**
   ```bash
   rm -f s3-policy.json s3-policy.json.bak
   git rm --cached s3-policy.json s3-policy.json.bak 2>/dev/null || true
   ```

3. **Sử dụng secrets thay vì hardcode:**
   - Tất cả credentials phải là GitHub Secrets
   - Không in secrets ra log (dùng `::add-mask::`)

4. **Xóa key pair cục bộ sau khi dùng xong:**
   ```bash
   rm ~/.ssh/$VM_KEY_NAME.pem ~/.ssh/income_deploy ~/.ssh/income_deploy.pub
   ```

---

Chúc bạn thành công!
