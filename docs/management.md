# Content

- [Logging](#logging)
- [Backup-Restore](#backup-restore)
  - [Backup](#backup)
  - [Restore](#restore)

===========================================================================

## <a name="logging">Logging</a>

Phần cấu hình logging aerospike nằm trong khối logging { ... } trong tệp cấu hình aerospike

Dưới đây là ví dụ cấu hình khối logging

```
logging {
	file /var/log/aerospike/aerospike.log {
		context any info
	}
}
```

logging gồm một số cấp độ như critical, warning, info, debug, detail

- Cấu hình logrotate

Thực hiện cấu hình rotate mỗi ngày, và duy trì tệp log cho 30 days

Tạo tệp tin /etc/logrotate.d/aerospike với nội dung sau:

```
/var/log/aerospike/aerospike.log {
    daily
    rotate 30
    dateext
    compress
    olddir /var/log/aerospike/
    sharedscripts
    postrotate
        /bin/kill -HUP `pidof asd`
    endscript
}

```

Yêu cầu tạo directory /var/log/aerospike trước khi thực hiện restart aerospike

Test logrotate `logrotate -f -v /etc/logrotate.d/aerospike`

Tham khảo thêm> [https://www.aerospike.com/docs/operations/configure/log/logrotate.html](https://www.aerospike.com/docs/operations/configure/log/logrotate.html)

## <a name="backup-restore">Backup & Restore</a>

Mặc định thì trong cụm cluster aerospike cũng là một dạng backup. Nhưng trong nhiều trường hợp như dữ liệu thay đổi, xóa thì dữ liệu cần được backup để phục hồi nếu cần.

### <a name="backup">Backup</a>

Tiện ích asbackup cho phép backup namespace và set từ một aerospike cluster đến vị trí lưu trữ local. 

- Backup to directory

cú pháp: `asbackup --host nodeIpAddressOrName --namespace namespaceName --directory pathOfDirectoryForBackupFiles`

Thông thường chúng ta backup theo cấu trúc thư mục lưu trữ ../ngay-thang/namespace/

ví dụ:

`asbackup --host 192.168.1.100 --namespace bar --directory /backup/20200601/bar`

Với backup chỉ định thư mục thì dữ liệu sẽ được lưu ở nhiều file với mở rộng là .asb và giới hạn mỗi file là 250MB.

- Backup to single file

cú pháp: `asbackup --host nodeIpAddressOrName --namespace namespaceName --output-file nameOfBackupFile`

ví dụ:

`asbackup --host 192.168.1.100 --namespace bar --output-file /backup/bar.asb`

Sử dụng tùy chọn '-' như tên tệp output khi đó có thể kết hợp pipeline cho output

ex:

`asbackup --host 192.168.1.100 --namespace bar --output-file - |gzip -1 >/backup/bar-20200601.asb.gz`

- Incremental Backup

Sử dụng tùy chọn `--modified-after` với date and time để backup incremental, với định dạng <YYYY-MM-DD_HH:MM:SS>

Ví dụ muốn backup dữ liệu của namespace `bar` thay đổi từ 2020-06-01_08:00:00 đến hiện tại, thực hiện như sau:

`asbackup --host 192.168.1.100 --modified-after 2020-06-01_08:00:00 --namespace bar --output-file - /backup/bar-current.asb`

### <a name="restore">Restore</a>

Cú pháp restore cơ bản một namespace từ node trong cluster

`asrestore --host nodeIpAddressOrName --directory pathOfDirectoryOfBackupedFiles`

ví dụ:

`asrestore --host 192.168.1.100 --directory /backup/20200601/bar`

- Restore từ một tệp backup

`asrestore --host nodeIpAddressOrName --input-file pathOfBackupFile`

ví dụ:

`asrestore --host 192.168.1.100 --input-file /backup/bar.asb`

- Restore đến namespace khác

`asrestore --host nodeIpAddressOrName --directory pathOfDirectoryOfBackupedFiles --namespace oldNamespaceName,newNamespaceName`

Readmore >>[https://www.aerospike.com/docs/tools/backup/](https://www.aerospike.com/docs/tools/backup/)
