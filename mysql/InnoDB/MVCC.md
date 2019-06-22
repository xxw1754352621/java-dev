# InnoDB的MVCC实现

- 快照读(snapshot read)

  简单的select操作(不包括 select ... lock in share mode, select ... for update)

- 当前读(current read)

  select ... lock in share mode

  select ... for update

  insert

  update

  delete