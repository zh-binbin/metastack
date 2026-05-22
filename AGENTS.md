# AGENTS.md

## Cursor Cloud specific instructions

This is **MetaStack**, a C-based fork of Slurm Workload Manager (v24.05.8). It is a HPC cluster job scheduler.

### Build

Standard GNU Autotools build (`configure` script is pre-generated):

```bash
./configure --enable-multiple-slurmd --prefix=/tmp/slurm --sysconfdir=/tmp/slurm/etc --disable-slurmrestd
make -j$(nproc)
make install
```

- `--disable-slurmrestd` is required because `openapi.json` files are not tracked in this repo, causing build failure in `src/slurmrestd/plugins/openapi/v0.0.39/`.
- Binaries install to `/tmp/slurm/` (sbin: `slurmctld`, `slurmd`, `slurmdbd`, `slurmstepd`; bin: `srun`, `sbatch`, `squeue`, `sinfo`, etc.).

### Running the cluster (single-node dev mode)

1. **Munge** (authentication): create key with `sudo dd if=/dev/urandom bs=1 count=1024 of=/etc/munge/munge.key`, set ownership to `munge:munge`, then start with `sudo -u munge munged --force`.
2. **Slurm config**: write `/tmp/slurm/etc/slurm.conf` and `/tmp/slurm/etc/cgroup.conf`. Key gotcha: in container/VM environments without systemd, set `CgroupPlugin=disabled` in `cgroup.conf` and use `ProctrackType=proctrack/linuxproc` in `slurm.conf`.
3. **Start daemons**: `slurmctld -D &` (as current user), `sudo slurmd -D &` (needs root for compute node daemon).

### Tests

- Unit tests: `cd testsuite/slurm_unit && make check` (11 tests, uses libcheck).
- Build warnings act as lint (compile with `-Wall`).
- Integration tests in `testsuite/expect/` use TCL Expect framework.
- Python tests in `testsuite/python/` use pytest-style scripts.

### Key gotchas

- The `CgroupPlugin=disabled` setting in `cgroup.conf` is critical for running in Docker/container environments without systemd. Without it, `slurmd` will fatal with "systemd scope for slurmstepd could not be set."
- `SlurmdUser=root` is typical; `slurmd` usually runs as root to manage processes.
- Add `/tmp/slurm/bin` and `/tmp/slurm/sbin` to `PATH` to use Slurm commands.
