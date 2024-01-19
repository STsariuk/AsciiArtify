# Installation instruction

1. Clone current repository `git clone https://github.com/STsariuk/AsciiArtify.git`.
2. Move to the script directory `./scripts/`.
3. Make scrip executable `chmod +x kubeplugin`.
4. Test the script `./kubeplugin kube-system`.

```bash
~/projects/AsciiArtify/scripts$ ./kubeplugin kube-system
Pod, kube-system, coredns-77ccd57875-v7k7x, 2m, 32Mi
Pod, kube-system, local-path-provisioner-957fdf8bc-mgb6x, 1m, 15Mi
Pod, kube-system, metrics-server-648b5df564-9hmhd, 4m, 42Mi
Pod, kube-system, svclb-traefik-314cd685-p6z2x, 0m, 0Mi
Pod, kube-system, traefik-64f55bb67d-7dxww, 1m, 43Mi
```

5. Add plugin to the `kubectl plugin`.

```bash
~/projects/AsciiArtify/scripts$ sudo mv ./kubeplugin /usr/local/bin/kubectl-kubeplugin
total 18444
drwxr-xr-x  2 root root     4096 Jan 19 15:19 ./
drwxr-xr-x 11 root root     4096 Nov  8 10:54 ../
-rwxr-xr-x  1 root root 18874368 Nov  6 15:35 k3d*
lrwxrwxrwx  1 root root       55 Jan  4 18:19 kubectl -> /mnt/wsl/docker-desktop/cli-tools/usr/local/bin/kubectl*
-rwxr-xr-x  1 root root      606 Jan 19 15:16 kubectl-kubeplugin*
```

6. Test plugin

```bash
~/projects/AsciiArtify/scripts$ kubectl kubeplugin demo
Pod, demo, ambassador-5f859c79c9-5fgmw, 8m, 118Mi
Pod, demo, cache-858575fc54-svqwg, 1m, 4Mi
Pod, demo, db-7968646c85-zxsw4, 4m, 40Mi
Pod, demo, demo-api-5479bcf989-dqb8b, 1m, 15Mi
Pod, demo, demo-ascii-548b45d685-mmhkh, 1m, 18Mi
Pod, demo, demo-data-78b5b48794-n2zfv, 1m, 18Mi
Pod, demo, demo-front-548899b579-qq8hr, 1m, 14Mi
Pod, demo, demo-img-5fd4bdf8bd-8dhkf, 1m, 17Mi
Pod, demo, demo-nats-0, 1m, 35Mi
Pod, demo, demo-nats-box-f9c9b584c-2k6jg, 1m, 0Mi
```