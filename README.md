## About aerospike

Aerospike là cơ sở dữ liệu NoQuery mã nguồn mở. Nó hỗ trợ mô hình lưu trữ key-value và document.

Kiến trúc aerospike gồm 3 lớp:

- Client layer

Aerospike client là lớp client nguồn mở. Nó tự động theo dõi cluster liên tục xem nên biết được truy cập dữ liệu từ vị trí nào trong một cụm cluster, tự động detect các transaction lỗi, và query lại ở một máy khác chứa cùng dữ liệu.

Download thư viện aerospike client tại [https://www.aerospike.com/download/client/](https://www.aerospike.com/download/client/)

- Clustering and Data Distribution Layer

Lớp này quản lý các giao tiếp cluster, tự động fail-over, replication, cross-datacenter replication (XDR), và cân bằng tải lại và di chuyển dữ liệu thông minh.

- Data Storage Layer

Lớp này chịu trách nhiệm lưu trữ dữ liệu trong DRAM và Flash cho phản hồi nhanh.

Aerospike là một lưu trữ key-value với mô hình dữ liệu schemaless. Dữ liệu được đẩy vào trong các hộp chứa, namespace policy. Namespace chia dữ liệu thành các tập(set) tương tự như table trong RDBMS và các bản ghi(record) tương ứng với row trong RDBMS. Mỗi record được đánh một index key duy nhất trong tập set và một hoặc nhiều hơn các tên bins tương ứng với column trong RDBMS.

## Setup Aerospike

- [Installing](https://github.com/keepwalking86/aerospike/blob/master/docs/setup.md#installing)

- [Configuration](https://github.com/keepwalking86/aerospike/blob/master/docs/setup.md#configuration)

## Management

- [Logging](https://github.com/keepwalking86/aerospike/blob/master/docs/management.md#logging)
- [Backup & Restore](https://github.com/keepwalking86/aerospike/blob/master/docs/management.md#backup-restore)

## Benchmark
