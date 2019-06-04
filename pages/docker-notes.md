# Some Notes from My Practice of Docker

# COPY
```
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

Dockerfiles的`COPY`指令不会创建原本不存在的目录，所以如果destination原本不存在，`COPY`指令不会做任何事。

`COPY`指令的source是一个目录的时候，并不会把整个source目录复制到destination下，而是把source目录里面的内容复制到destination下。

# RUN
(以我现在用Docker的阶段)执行多条指令时不要写多个`RUN`，应该写一个`RUN`然后用反斜杠`\`来换行