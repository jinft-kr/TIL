### [Issue]
- GCP GCE에 bastion host를 만들고, Cloud SQL internal IP로 Database에 접근하고자 하였다.
- 따라서 GCP GCE에 User를 생성 하였는데, ssh 접속 시 `Permission denied (publickey)` error 발생하였다.
- 또는 Mysql Workbench `Standard TCP/IP over SSH` connection method를 통해 bastion host에 접근하는데, `Cloud not connect the SSH Tunnel`Error 가 발생하였다.
- ![Mysql_Workbench_ssh_connection_error](https://user-images.githubusercontent.com/63401132/176194321-e5517c12-9278-4263-af32-f61ba3d3b06d.jpeg)

---

### [Problem Solution Approach]
- GCE meta-data에 `enable-oslogin = TRUE` 추가
- 내 계정에 필요한 IAM role(아래 role 중 하나만 있으면 됨)
  - `roles/compute.osLogin` - 관리자 권한을 부여하지 않음
  - `roles/compute.osAdminLogin` - 관리자 권한을 부여함
- `gcloud auth login` : 구글 계정으로 로그인
- `gcloud` 명령어로 GCE에 IAM 권한으로 접속
  - `gcloud compute ssh --zone ${zone} ${gsc_id} --project ${project_id}
  - ![gcloud_compute_ssh](https://user-images.githubusercontent.com/63401132/176195759-cd8123e5-8340-4ac8-a3bb-24e9080d8ee3.jpeg)
- `/etc/ssh/sshd_config` 파일 수정 : `vi /etc/ssh/sshd_config`
  - `PasswordAuthentication no` → `PasswordAuthentication yes`
  - ![password_authentication](https://user-images.githubusercontent.com/63401132/176196067-78f53919-7a2b-4656-b0e6-9ea7ee5c8e9a.jpeg)
- `sudo service ssh restart ssh`

---

### [Concept]
- [Set up OS Login](https://cloud.google.com/compute/docs/oslogin/set-up-oslogin)
- [How to connect to GCP VM instance with password using SSH?](https://stackoverflow.com/questions/59533620/how-to-connect-to-gcp-vm-instance-with-password-using-ssh)
