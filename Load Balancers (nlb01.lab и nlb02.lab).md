# cloud config

```
#cloud-config
hostname: nlb01.lab
runcmd: 
- date > /opt/date

#cloud-config
hostname: nlb02.lab
runcmd: 
- date > /opt/date
```
# обновляем пакеты
```bash
sudo dnf update -y  
```

# Nginx
## установка Nginx через EPEL
```bash
sudo dnf install epel-release -y
sudo dnf install nginx -y
```
