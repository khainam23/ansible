# Ansible Collection

**Collection** = package chứa `roles`, `plugins`, `modules`, `playbooks`... để tái sử dụng và version hóa.

### Tạo

```bash
ansible-galaxy collection init company.infra
```

### Build

```bash
ansible-galaxy collection build
```

→ `company-infra-1.0.0.tar.gz`

Version nằm trong `galaxy.yml`.

### Cài từ local `.tar.gz`

```yaml
collections:
  - name: ../collections/company-infra-1.0.0.tar.gz
    type: file
```

```bash
ansible-galaxy collection install -r requirements.yml
```

### Cài từ Galaxy

```yaml
collections:
  - name: company.infra
    version: "1.0.0"
```

### Dùng role

```yaml
- hosts: all
  roles:
    - company.infra.myrole
```

### Team workflow

```text
Role → Collection → build .tar.gz → Private Registry → requirements.yml
```

**Nhớ:** `Role` = logic, `Collection` = package, `requirements.yml` = dependency.
