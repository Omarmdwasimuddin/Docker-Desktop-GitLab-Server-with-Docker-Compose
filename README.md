# Docker-Desktop: GitLab server with docker compose

#### Powershell open koro command daw
```bash
mkdir Demo
cd Demo
code .
```
---

#### DEMO Project e file create koro: docker-compose.yml
```bash
version: '5.4'
services:
  gitlab-server:
    image: 'gitlab/gitlab-ce'
    container_name: my-gitlab-server
    ports:
      - '8000:80'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        gitlab_rails['initial_root_password'] = 'w@sIm1997'
        puma['worker_processes'] = 0 
```
---

#### vs terminal e command daw
```bash
docker compose up
```
---

#### browser e search koro: http://localhost:8000/users/sign_in
#### Note: kichukhon somoy nibe
#### Username daw: root, Password daw: w@sIm1997
#### er por setup options asbe setup korle dashbord e niye jabe.
---

#### vs terminal e command daw
```bash
docker compose down
```
---
