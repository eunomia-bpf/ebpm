# statsnoop

- version: 0.1.0
- license: GPL,
- author: John Doe

Trace stat syscalls.

Run:

```shell
sudo ./ecli statsnoop
```

> this should be auto generated.

## result

origin from:

https://github.com/iovisor/bcc/blob/master/libbpf-tools/statsnoop.bpf.c

## Run

(just replace the path as yours)

Compile:

```shell
docker run -it -v /home/yunwei/coding/eunomia-bpf/bpftools/examples/opensnoop:/src yunwei37/ebpm:latest
```

Run:

```shell
sudo ./ecli run bpftools/examples/opensnoop/package.json
```