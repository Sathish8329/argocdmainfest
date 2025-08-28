✅ 1. Get your AWS account ID
aws sts get-caller-identity --query "Account" --output text


Assume it returns:
ACCOUNT_ID=123456789012

✅ 2. Create ECR repository
aws ecr create-repository \
  --repository-name myapp \
  --region ap-south-1

✅ 3. Authenticate Docker to ECR and push image
# Get login token and authenticate Docker
aws ecr get-login-password --region ap-south-1 \
  | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com

# Build your Docker image
docker build -t myapp:v1 .

# Tag with full ECR URI
docker tag myapp:v1 123456789012.dkr.ecr.ap-south-1.amazonaws.com/myapp:v1

# Push to ECR
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/myapp:v1

✅ 4. Find node group role and attach ECR pull policy

Get your node group name(s):

aws eks list-nodegroups --cluster-name kovaionai --region ap-south-1


Assume the node group name is kovaionai-nodegroup.

Find IAM role for the node group:

aws eks describe-nodegroup \
  --cluster-name kovaionai \
  --nodegroup-name kovaionai-nodegroup \
  --query "nodegroup.nodeRole" \
  --output text


This gives something like:

arn:aws:iam::123456789012:role/eks-node-role


Extract the role name: eks-node-role.

Attach ECR read-only policy:

aws iam attach-role-policy \
  --role-name eks-node-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly


✅ This allows all nodes in that group to pull ECR images without storing keys or secrets.
