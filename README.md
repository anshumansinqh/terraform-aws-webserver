

-> Step 1 — AWS CLI installed and configured
<img width="1566" height="553" alt="01-aws-credentials png" src="https://github.com/user-attachments/assets/daddae13-b9dc-4122-a43d-f75f3ba78b58" />


- Opened Windows PowerShell on Windows 11
- Ran `aws --version` → confirmed AWS CLI v2.34.55 is installed
- Ran `aws configure` and entered:
  - AWS Access Key ID: `AKIA455F57O6XWGFFX5U`
  - AWS Secret Access Key: (entered secret key)
  - Default region: `us-east-1`
  - Default output format: `json`
- Ran `aws sts get-caller-identity` to verify credentials
- Output confirmed:
  - UserId: `AIDA455F57O63PYOJ4A3G`
  - Account: `888869354429`
  - ARN: `arn:aws:iam::888869354429:user/terraform-user`
- This confirmed Terraform has a valid, working AWS identity

---

-> Step 2 — IAM user verified in AWS console
<img width="1535" height="571" alt="02-iam-user png" src="https://github.com/user-attachments/assets/f7d6d261-a896-4c6b-a2e1-d1d1971d954b" />

- Logged into AWS Management Console
- Navigated to IAM → Users → `terraform-user`
- Clicked the **Security credentials** tab
- Confirmed:
  - ARN: `arn:aws:iam::888869354429:user/terraform-user`
  - Created: May 28, 2026 at 08:40 UTC+05:30
  - Console access: **Disabled** (intentional — CLI only)
  - Access Key 1: `AKIA455F57O6XWGFFX5U` — Status: **Active**
  - Console password: **Not enabled** (secure, no GUI login)
- This confirms the IAM user is properly set up for programmatic access only

---

-> Step 3 — Terraform installed and project folder created
<img width="1681" height="560" alt="03-terraform-version png" src="https://github.com/user-attachments/assets/6a6ea84f-e699-4cf2-b7dd-7d7686dbf69e" />


- Ran `terraform --version` → confirmed **Terraform v1.15.5** on windows_amd64
- Ran `New-Item -ItemType Directory -Path "C:\terraform-webserver"` → created project folder
- Output showed folder `terraform-webserver` created at `C:\` on 28-05-2026 at 09:01
- Ran `cd C:\terraform-webserver` to enter the folder
- Ran AWS CLI command to fetch the latest Amazon Linux 2 AMI ID dynamically:
  ```
  aws ec2 describe-images --owners amazon
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2"
            "Name=state,Values=available"
  --query "sort_by(Images, &CreationDate)[-1].ImageId"
  --output text
  ```
- Output: `ami-046b36f55e189564a` — this is the AMI ID used in `main.tf`

---

-> Step 4 — main.tf created
<img width="1696" height="603" alt="04-main-tf png" src="https://github.com/user-attachments/assets/30dbeff4-93c9-4e7c-ad5a-dfdcddb7997b" />


- Ran `notepad main.tf` to open Notepad and create the file
- Pasted the complete Terraform configuration into Notepad and saved
- Ran `dir` to verify the file exists
- Output confirmed:
  - File name: `main.tf`
  - Created: 28-05-2026 at 09:04
  - File size: **992 bytes**
  - Location: `C:\terraform-webserver\`
- The `main.tf` file contains: AWS provider block, security group resource, EC2 instance resource with user data script, and output block

---

-> Step 5 — terraform init executed
<img width="1394" height="596" alt="05-terraform-init png" src="https://github.com/user-attachments/assets/19794f73-b95d-4f1f-ad0c-424bbf1d01e0" />


- Ran `terraform init` inside `C:\terraform-webserver`
- Terraform found the AWS provider in the configuration
- Downloaded and installed **hashicorp/aws v6.47.0** (signed by HashiCorp)
- Initialized the backend successfully
- Created `.terraform.lock.hcl` file to lock provider version for future runs
- Final output: **"Terraform has been successfully initialized!"**
- This means Terraform is ready to communicate with the AWS API

---

-> Step 6 — terraform plan executed
<img width="1424" height="857" alt="06-terraform-plan png" src="https://github.com/user-attachments/assets/a293f4e9-b1d6-48b5-85a6-88cd13fe26c3" />


- Ran `terraform plan` to preview what will be created
- Terraform showed the full execution plan for `aws_instance.web_server`:
  - AMI: `ami-046b36f55e189564a`
  - Instance type: `t2.micro` (later changed to `t3.micro`)
  - All other values shown as `(known after apply)` — assigned by AWS at creation time
  - User data script confirmed present (yum install httpd etc.)
  - Tags: `Name = "TerraformWebServer"`
- Also showed `aws_security_group.web_sg` with:
  - Ingress: port 80, TCP, `0.0.0.0/0`
  - Egress: all traffic, `0.0.0.0/0`
- Final line: **"Plan: 2 to add, 0 to change, 0 to destroy"**
- No real infrastructure created yet — this is just a preview

---

-> Step 7 — terraform apply executed — server deployed
<img width="1150" height="493" alt="07-terraform-apply png" src="https://github.com/user-attachments/assets/4797d9db-efd8-4672-ae6c-2c8365823ccb" />


- Ran `terraform apply`
- Terraform showed the plan again and prompted for confirmation
- Typed `yes` to approve
- Terraform executed in sequence:
  - `aws_instance.web_server: Creating...`
  - `aws_instance.web_server: Still creating... [00m10s elapsed]`
  - `aws_instance.web_server: Creation complete after 17s` with ID `i-06fa313ddc5d7a4ec`
- Final output:
  - **"Apply complete! Resources: 1 added, 0 changed, 0 destroyed"**
  - `website_url = "http://98.94.45.39"` — the live public IP of the server

---

-> Step 8 — Live website verified in browser
<img width="1919" height="966" alt="08-live-website png" src="https://github.com/user-attachments/assets/b0eb746f-ccd4-4f81-9b66-c1e33803f437" />


- Opened browser and navigated to `http://98.94.45.39`
- Website loaded successfully displaying:
  - **"Hello from Terraform!"** (H1 heading)
  - **"Deployed by me on AWS EC2."** (paragraph)
- URL bar shows `98.94.45.39` with "Not secure" (HTTP, not HTTPS — expected)
- This confirms:
  - EC2 instance is running
  - Apache HTTP server is installed and active
  - User data script executed successfully at boot
  - Security group port 80 rule is working
  - Website is publicly accessible from anywhere in the world

---

-> Step 9 — terraform destroy executed
<img width="1149" height="507" alt="09-terraform-destroy png" src="https://github.com/user-attachments/assets/72ad7fd8-3ebd-489a-a3ea-c56741a8cb0a" />


- Ran `terraform destroy` to remove all AWS resources
- Terraform showed the plan: **"Plan: 0 to add, 0 to change, 2 to destroy"**
- Output showed: `website_url = "http://98.94.45.39" → null`
- Prompted: "Do you really want to destroy all resources?" → typed `yes`
- Terraform destroyed in sequence:
  - `aws_instance.web_server: Destroying... [id=i-06fa313ddc5d7a4ec]`
  - Still destroying at 10s, 20s, 30s elapsed
  - `aws_instance.web_server: Destruction complete after 33s`
  - `aws_security_group.web_sg: Destroying... [id=sg-0ca8a1d70c87dd41d]`
  - `aws_security_group.web_sg: Destruction complete after 1s`
- Final output: **"Destroy complete! Resources: 2 destroyed"**
- AWS cost goes back to $0

---

-> Step 10 — AWS console confirmation
<img width="1609" height="367" alt="10-aws-console png" src="https://github.com/user-attachments/assets/1b82ed38-37ad-4427-8498-5c73dcf94fba" />


- Logged into AWS Console → EC2 → Instances → switched to **us-east-1** region
- Console shows **2 instances** in history:
  - `TerraformWeb...` — ID: `i-06fa313ddc5d7a4ec` — State: **Terminated** — t3.micro — us-east-1c (first deployment, now destroyed)
  - `TerraformWeb...` — ID: `i-02b96a06ff0eca689` — State: **Running** — t3.micro — us-east-1c — DNS: `ec2-54-91-249-27.com...` (second deployment)
- This visually proves that `terraform apply` and `terraform destroy` work cleanly and repeatedly
- Both instances are in **us-east-1c** availability zone as configured

---

Now copy all these points and add them to your README under each screenshot section. This gives your GitHub repo a very detailed, professional, and honest documentation of exactly what you did. Want me to put this all together into the final README format for you?
