.db-shm
    Shared Memory
    SHM 파일은 WAL 파일을 사용할 때 메모리 상태를 공유하기 위한 용도로 사용됩니다
.db-wal
    Write-Ahead Logging
    https://www.sqlite.org/wal.html
    WAL 모드에서는 데이터베이스에 변경 사항을 직접 기록하는 대신, WAL 파일에 먼저 기록합니다. 이후, 일정 조건이 충족되면 WAL 파일의 내용을 메인 데이터베이스 파일로 병합하는 과정을 거칩니다
    WAL 모드에서는 쓰기 작업이 WAL 파일에 기록되므로 읽기 작업이 차단되지 않습니다. 이를 통해 쓰기 성능과 읽기 동시성이 크게 향상됩니다.


- https://stackoverflow.com/questions/7778723/what-are-the-db-shm-and-db-wal-extensions-in-sqlite-databases