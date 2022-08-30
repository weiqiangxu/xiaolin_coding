# Redis对象和底层数据结构关系

1. String 底层 SDS
2. List 底层是 QuickList
3. Hash 底层是 ZipList && HashTable
4. Set 底层是 HashTable
5. Sorted Set 底层是ZskipList