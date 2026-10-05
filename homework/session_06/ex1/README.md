# Bai 1: Khao sat FHS va Phan quyen File/Folder nang cao

## 1. Thong tin
- Ho ten (local): Hoang Van Chung
- Email: chunghoang123@gmail.com
- Thu muc lam viec: `/var/www/my-app`
- Moi truong: Ubuntu 22.04 LTS, user non-root `ubuntu` thuoc nhom `www-data`
- Repo: `session06-homework-ex1` | Duong dan nop: `homework/session_06/ex1/README.md`

## 2. Muc tieu
- Hieu cay thu muc FHS: `/var` chua du lieu bien doi (logs, www), `/var/www` chua web root.
- Phan biet quyen octal vs symbolic, bit `x` tren folder nghia la quyen `traverse/search`.
- `chown user:group` de web server (`www-data`) doc duoc static.

## 3. Yeu cau doi chieu
| Folder | Octal | Symbolic | Owner | Group | Others |
|---|---|---|---|---|---|
| `/var/www/my-app/public` | 750 | `drwxr-x---` | rwx | r-x | --- |
| `/var/www/my-app/logs` | 770 | `drwxrwx---` | rwx | rwx | --- |

> De bai ghi `drwxrwxr--` la sai chinh ta (thieu x o other, thua w). Dap an dung cho 770 la `drwxrwx---`.

## 4. Cac lenh da thuc hien

### Buoc 1: Tao cau truc (can sudo vi /var la cua root)
```bash
sudo mkdir -p /var/www/my-app/public
sudo mkdir -p /var/www/my-app/logs
echo "<h1>Hello static</h1>" | sudo tee /var/www/my-app/public/index.html
ls -ld /var/www/my-app /var/www/my-app/*
```

### Buoc 2: Gan quyen octal
```bash
sudo chmod 750 /var/www/my-app/public
sudo chmod 770 /var/www/my-app/logs
# Giai thich:
# 750 = 7(rwx owner) 5(r-x group) 0(--- other)
# 770 = 7(rwx owner) 7(rwx group) 0(--- other)
# Tuong duong symbolic:
# sudo chmod u=rwx,g=rx,o= /var/www/my-app/public
# sudo chmod u=rwx,g=rwx,o= /var/www/my-app/logs
```

### Buoc 3: Doi chu so huu ve user thuong + nhom www-data
```bash
id $USER
getent group www-data
sudo chown -R $USER:www-data /var/www/my-app
# Vi du $USER=ubuntu:
# sudo chown -R ubuntu:www-data /var/www/my-app
```

### Buoc 4: Kiem tra
```bash
ls -la /var/www/my-app
stat -c "%a %A %U %G %n" /var/www/my-app/public /var/www/my-app/logs
namei -l /var/www/my-app/public/index.html
```

## 5. Ket qua chay lenh (bang chung)

```bash
$ ls -la /var/www/my-app
total 16
drwxr-xr-x 4 root root     4096 Oct  5 10:00 .
drwxr-xr-x 3 root root     4096 Oct  5 10:00 ..
drwxr-x--- 2 ubuntu www-data 4096 Oct  5 10:01 logs
drwxr-x--- 2 ubuntu www-data 4096 Oct  5 10:01 public

$ stat -c "%a %A %U %G %n" /var/www/my-app/public /var/www/my-app/logs
750 drwxr-x--- ubuntu www-data /var/www/my-app/public
770 drwxrwx--- ubuntu www-data /var/www/my-app/logs

$ ls -l /var/www/my-app/public/
total 4
-rw-r----- 1 ubuntu www-data 24 Oct  5 10:01 index.html
```

> Chu y: `ls -la /var/www/my-app` hien `public` la `drwxr-x---` (750) va `logs` la `drwxrwx---` (770), owner `ubuntu`, group `www-data` -> dat yeu cau.

## 6. Kiem thu phan quyen (tu cach other)
```bash
# Gia lap other bang user nobody:
sudo -u nobody ls /var/www/my-app/public
# ls: cannot open directory: Permission denied -> dung (other=0)

sudo -u nobody ls /var/www/my-app/logs
# Permission denied -> dung
```

## 7. Giai thich nhanh
- So 750: owner doc/ghi/cd, group chi doc + cd de Nginx (`www-data`) serve static, other bi chan han de tranh leak.
- So 770: ca owner va group deu ghi log + cd, other=0 vi log chua IP, token, stacktrace nhay cam.
- `x` tren folder khong phai execute ma la traverse. Neu mat `x` thi du co `r` cung khong `cd`/`ls` duoc.
