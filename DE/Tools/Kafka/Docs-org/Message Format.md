Các messages (hay còn gọi là bản ghi) luôn được ghi theo lô. Thuật ngữ kỹ thuật cho một lô message là lô bản ghi, và một lô bản ghi chứa một hoặc nhiều bản ghi. Trong trường hợp suy biến, chúng ta có thể có một lô bản ghi chỉ chứa một bản ghi duy nhất. Lô bản ghi và bản ghi đều có tiêu đề riêng. Định dạng của mỗi loại được mô tả bên dưới.

## Record Batch

Dưới đây là định dạng dữ liệu trên đĩa của một RecordBatch.

```text
baseOffset: int64
batchLength: int32
partitionLeaderEpoch: int32
magic: int8 (current magic value is 2)
crc: uint32
attributes: int16
    bit 0~2:
        0: no compression
        1: gzip
        2: snappy
        3: lz4
        4: zstd
    bit 3: timestampType
    bit 4: isTransactional (0 means not transactional)
    bit 5: isControlBatch (0 means not a control batch)
    bit 6: hasDeleteHorizonMs (0 means baseTimestamp is not set as the delete horizon for compaction)
    bit 7~15: unused
lastOffsetDelta: int32
baseTimestamp: int64
maxTimestamp: int64
producerId: int64
producerEpoch: int16
baseSequence: int32
recordsCount: int32
records: [Record]
```

### Control Batches

Một control batch chứa một bản ghi duy nhất được gọi là control record. Control records không nên được chuyển tiếp đến các ứng dụng. Thay vào đó, chúng được người dùng sử dụng để lọc ra các transactional messages bị hủy bỏ.

```text
version: int16 (current version is 0)
type: int16 (0 indicates an abort marker, 1 indicates a commit)
```

## Record

Định dạng dữ liệu trên đĩa của mỗi bản ghi được mô tả chi tiết bên dưới.

```text
length: varint
attributes: int8
    bit 0~7: unused
timestampDelta: varlong
offsetDelta: varint
keyLength: varint
key: byte[]
valueLength: varint
value: byte[]
headersCount: varint
Headers => [Header]
```

### Record Header

```text
headerKeyLength: varint
headerKey: String
headerValueLength: varint
Value: byte[]
```

## Old Message Format

