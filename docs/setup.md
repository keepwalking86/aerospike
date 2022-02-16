# Content

- [Installing](#installing)
- [Configuration](#configuration)
  - [Network](#network)
  - [Namespace](#namespace)
  - [Cluster](#cluster)

============================================================

## <a name="installing">Installing</a>

- Download

`wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/centosVersion'`

ex:

`wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/el7'`

current version: `https://www.aerospike.com/download/server/5.0.0.4/artifact/el7`

- Uncompress

`tar -xvf aerospike.tgz`

- Installing

Tùy thuộc vào version hiện tại, chọn đường dẫn directory phù hợp

ex:

```
cd aerospike-server-community-5.0.0.4-el7/
./asinstall
```

- Start aerospike as systemd

```
systemctl enable aerospike
systemctl start aerospike
```

## <a name="configuration">Configure Aerospike</a>

### <a name="network">Network Configuration</a>

Dưới đây là một số cổng mặc định cho các chức năng khác nhau trong Aerospike

- Port service (3000): Các application, tool và remote XDR thao tác cơ sơ và trạng thái cluster của aerospike qua port này

- Port Fabric (3001): Port giao tiếp giữa các cluster. Các thao tác replication write, migration, và giao tiếp node-to-node sử dụng port này

- Port Mesh heartbeat (3002): Port này được tạo khi hình thành và duy trì cluster và giao tiếp unicast. Chỉ có một port heartbeat duy nhất được cấu hình trong cluster.

- Port Multicast Heartbeat (9918): Port này được tạo khi hình thành và duy trì cluster và giao tiếp multicast. Chỉ có một port heartbeat duy nhất được cấu hình trong cluster.

- Port info (3003): Port này dùng cho quản trị một số thông tin lệnh đơn giản.


**Cấu hình**

Mặc định khi cài đặt xong khối cấu hình network có cấu hình như sau:

```
network {
        service {
                address any
                port 3000
        }

        heartbeat {
                mode multicast
                multicast-group 239.1.99.222
                port 9918

                # To use unicast-mesh heartbeats, remove the 3 lines above, and see
                # aerospike_mesh.conf for alternative.

                interval 150
                timeout 10
        }

        fabric {
                port 3001
        }

        info {
                port 3003
        }
}
```

### <a name="namespace">Namespace Configuration</a>

Đường dẫn tệp cấu hình mặc định **/etc/aerospike/aerospike.conf**

Cú pháp cấu hình đơn giản một namespace trong aerospike

```
namespace <namespace-name> {
    # memory-size 4G           # 4GB of memory to be used for index and data
    # replication-factor 2     # For multiple nodes, keep 2 copies of the data
    # high-water-memory-pct 60 # Evict non-zero TTL data if capacity exceeds
                               # 60% of 4GB
    # stop-writes-pct 90       # Stop writes if capacity exceeds 90% of 4GB
    # default-ttl 0            # Writes from client that do not provide a TTL
                               # will default to 0 or never expire
    # storage-engine memory    # Store data in memory only
}
```

Ở đây có một số tham số tùy chọn cần biết:

- memory-size: lượng kích thước memory cấp phát được sử dụng cho index và data

- replication-factor: Cấp phát số bản copy dữ liệu cho cluster nhiều node

- storage-engine: kỹ thuật lưu trữ cố định dữ liệu. Có 03 mode lưu trữ device, memory và pmem(chỉ có ở bản enteprise)

Với kỹ thuật lưu trữ **device** thì dữ liệu write đến node này sẽ được cố định trong một device hoặc file raw. Kỹ thuật này chỉ dùng cho SSD; Với kỹ thuật lưu trữ memory thì dữ liệu chỉ được write vào DRAM. Với pmem thì dữ liệu được cố định trong memory luôn.

Một số ví dụ cho các kỹ thuật lưu trữ cố định dữ liệu trên aerospike

```
namespace bar {
        replication-factor 2
        memory-size 4G

        storage-engine memory #only store data in memory
}
```

```
namespace vkid {
    memory-size 4G         # Maximum memory allocation for primary
                                # and secondary indexes.
    storage-engine device {     # Configure the storage-engine to use persistence
        device /dev/sdb    # raw device. Maximum size is 2 TiB
        device /dev/sdc  # (optional) another raw device.
        write-block-size 128K   # adjust block size to make it efficient for SSDs.
    }
}
```

```
namespace bar {
        replication-factor 2
        memory-size 4G

        storage-engine device {
                file /opt/aerospike/bar.dat # Location of data file on server
                filesize 50G # Max size of each file in GiB
                data-in-memory true # Indicates that all data should also be in memory.
		#data-in-memory false # Store data in memory in addition to file.
        }
}
```

### <a name="cluster">Cluster

Cấu hình với mô hình đơn giản gồm 03 node trong cluster: 10.10.10.10,10.10.10.11 và 10.10.10.12

Cấu hình cluster với mode mesh (unicast)

- Cấu hình network

```
network {
        service {
                address any
                port 3000
        }

        heartbeat {
                mode mesh
                port 3002
                address 10.10.10.10
                mesh-seed-address-port 10.10.10.10 3002
                mesh-seed-address-port 10.10.10.11 3002
		mesh-seed-address-port 10.10.10.12 3002

                interval 150
                timeout 10
        }

        fabric {
                port 3001
        }

        info {
                port 3003
        }
}
```

Khi đó tệp cấu hình aerospike.conf như sau:

```
service {
	paxos-single-replica-limit 1 # Number of nodes where the replica count is automatically reduced to 1.
	proto-fd-max 15000
}

logging {
    file /var/log/aerospike/aerospike.log {
        context any info
    }
}

network {
        service {
                address any
                port 3000
        }

        heartbeat {
                mode mesh
                port 3002
                address 10.10.10.10
                mesh-seed-address-port 10.10.10.10 3002
                mesh-seed-address-port 10.10.10.11 3002
		mesh-seed-address-port 10.10.10.12 3002

                interval 150
                timeout 10
        }

        fabric {
                port 3001
        }

        info {
                port 3003
        }
}

namespace test {
	replication-factor 2
	memory-size 4G

	storage-engine memory
}

namespace bar {
	replication-factor 2
	memory-size 4G

	storage-engine memory
}
```

Khi start aerospike service và sử dụng các công cụ như `asadm` kiểm tra thông cluster
