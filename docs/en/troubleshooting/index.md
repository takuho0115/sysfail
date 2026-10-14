# Troubleshooting {#troubleshooting}

## Run `sysfail doctor` first {#doctor}

`sysfail doctor` checks that the software and settings the game needs are in place. Run it once before playing.

- `[OK]` means fine, `[!!]` is a warning (the game still starts), and `[NG]` means a requirement for starting is not met.
- A line with a problem shows a one-line remedy and the URL of the item's help page ("More"). The steps and command examples are on that page. Offline, the one-line remedy tells you what to fix.
- By default, `sysfail doctor` skips the container-based checks (the isolation test and the daemon-side kernel checks). Run `sysfail doctor --full` for a detailed check. `sysfail doctor --build` first builds any missing game images and then runs the same checks as `--full` (first time only: 5 to 20 minutes, about 4 GB; if the docker data area has less than 6 GB free, it skips the build and shows how to free space).
- For scripts, the exit code is 3 if there is at least one `[NG]`, otherwise 0.

## doctor items {#doctor-items}

- [PF-01 docker command](pf-01.md)
- [PF-02 docker daemon](pf-02.md)
- [PF-03 docker kind](pf-03.md)
- [PF-04 docker Engine version](pf-04.md)
- [PF-05 docker compose](pf-05.md)
- [PF-06 tmux](pf-06.md)
- [PF-07 Container runtime version](pf-07.md)
- [PF-08 docker data free space](pf-08.md)
- [PF-09 docker VM memory](pf-09.md)
- [PF-10 Supported environment](pf-10.md)
- [PF-11 docker endpoint](pf-11.md)
- [PF-12 Network address range](pf-12.md)
- [PF-13 Firewall method](pf-13.md)
- [PF-14 Host listeners](pf-14.md)
- [PF-15 Bridge netfilter](pf-15.md)
- [PF-16 Leftover containers](pf-16.md)
- [PF-17 Isolation test](pf-17.md)
- [PF-18 Kernel modules](pf-18.md)
- [PF-19 Game images](pf-19.md)
