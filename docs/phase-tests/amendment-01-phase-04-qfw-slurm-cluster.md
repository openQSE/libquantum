# Amendment 01 Phase 4 QFw-SLURM-Cluster Qualification

This procedure qualifies the installable libquantum packages against the
deployed QFw, qfw-slurm, and QFw-SLURM-Cluster paths. It covers the Amendment
01 dual-backend system gate and the Amendment 02 compiler deployment gate.

The retained runs use NWQSim. They are simulator evidence only and provide no
physical-device qualification.

## Frozen revisions and packages

- libquantum `f0b5f5c`, package `0.1.0.dev0`
- libquantum-compiler-qiskit `b1d19e927803`, package `0.1.0`
- libquantum-compiler-mqt `aa0842bfdbdb`, package `0.1.0`
- QFw, including the Qiskit adapter, QPM, and qhw helpers,
  `91be0828ea91`
- qfw-slurm `e940e0a60022`
- QFw-SLURM-Cluster build/system gate `17ec601`
- QFw-SLURM-Cluster managed-QFw pytest gate `13e4d83`
- Qiskit environment `qiskit==2.2.3`
- MQT environment `mqt-core==4.1.1`, `qiskit==2.5.2`

The QFw tree supplies the NWQSim QPM query, allocation-scoped capacity,
telemetry records, `rz` parameter metadata, target helpers, and QRI execution
binding. The QPM advertises OpenQASM 2.0 as its accepted exchange input.

## Build and install

Mount all listed repositories below `/workspace/qfw-container-base`. Build the
exact QFw prefix and the two separate Python environments from the cluster
repository:

```bash
cd /path/to/QFw-SLURM-Cluster
QFW_DEV_SRC=/workspace/qfw-container-base/QFw-release-v0.1/\
repos-main-sync/QFw \
QFW_DEV_BUILD=/workspace/qfw-container-base/qfw-main-build \
QFW_DEV_PREFIX=/workspace/qfw-container-base/qfw-main-install \
QFW_DEV_VENV=/workspace/qfw-container-base/qfw-main-venv \
LIBQUANTUM_DEV_SRC=/workspace/qfw-container-base/libquantum \
QISKIT_COMPILER_DEV_SRC=/workspace/qfw-container-base/\
libquantum-compiler-qiskit \
MQT_COMPILER_DEV_SRC=/workspace/qfw-container-base/\
libquantum-compiler-mqt \
QFW_SLURM_DEV_SRC=/workspace/qfw-container-base/qfw-slurm \
./do_qfw_build.sh --clean
```

The build must print the installed libquantum path, both packaged
`compiler.json` paths, the qfw-slurm gateway import, the qfw MQT helper import,
and the MQT Core path. The main environment must report Qiskit 2.2.3. The MQT
environment must report MQT Core 4.1.1 and Qiskit 2.5.2.

Restart the site QPM with the exact prefix. Exporting `QFW_PREFIX` is required
so container defaults cannot silently select another installation.

```bash
docker exec \
  -e QFW_INSTALL_PREFIX=/workspace/qfw-container-base/qfw-main-install \
  -e QFW_VENV=/workspace/qfw-container-base/qfw-main-venv \
  slurmctld \
  /workspace/qfw-container-base/QFw-release-v0.1/repos-main-sync/\
QFw-SLURM-Cluster/tools/qfw-site-services restart --target nwqsim
```

## Slurm system runs

Submit both system scripts from the shared directory:

```bash
docker exec slurmctld bash -lc '
  cd /workspace/qfw-container-base/QFw-release-v0.1/repos-main-sync/\
QFw-SLURM-Cluster/shared-dir
  sbatch libquantum-mqt-qri.sbatch
  sbatch libquantum-dual-backend.sbatch
'
```

Wait for terminal state and inspect the retained logs:

```bash
docker exec slurmctld sacct \
  -j 37,41 --format=JobIDRaw,JobName,State,ExitCode,Elapsed
docker exec slurmctld sed -n '1,260p' \
  /workspace/home/user-a/libquantum-mqt-qri.37.out
docker exec slurmctld sed -n '1,360p' \
  /workspace/home/user-a/libquantum-dual-backend.41.out
```

Job 37 completed with exit code 0 in 21 seconds. It compiled a native
`mqt.core.mlir.QCProgram`, retained the QPM qhw target source and compiler
provenance, submitted operation
`1761d639-6973-46c5-acb8-cbe3de97384b`, and returned 126 `00` plus 130 `11`
counts for 256 shots.

Job 41 completed with exit code 0 in 34 seconds. It selected
`qfw_qiskit.qfw_simulator.QFwBackend` for `legacy` and
`qfw_qiskit.qfw_backend_v2.QFwBackendV2` for `v2`, with no fallback. Both ran
the same four-qubit GHZ circuit for 1024 shots through the allocation's
NWQSim reservation, returned only `0000` and `1111`, and returned a
statevector. NWQSim exposes no seed control, so qualification compares these
deterministic invariants instead of exact random count equality.

## Regression gates

Run the exact QFw legacy/V2 gates:

```bash
docker exec slurmctld bash -lc '
  cd /workspace/qfw-container-base/QFw-release-v0.1/repos-main-sync/QFw
  export LIBQUANTUM_SOURCE=/workspace/qfw-container-base/libquantum
  PYTHONPATH=/workspace/qfw-container-base/QFw-release-v0.1/\
repos-main-sync/QFw/backends:/workspace/qfw-container-base/QFw-release-v0.1/\
repos-main-sync/QFw/DEFw/python/infra:/workspace/qfw-container-base/libquantum \
  /workspace/qfw-container-base/qfw-main-venv/bin/python -m pytest -q \
  tests/qiskit tests/phase_tests/test_phase_19_qfw_backend_v2.py
'
```

The retained result is `98 passed`. The exact qfw-slurm gateway/protocol gate
reports `7 passed`. Core reports `124 passed, 1 skipped`; the skip is the
separate physical deployment gate and is not counted as evidence. The Qiskit
compiler package reports `11 passed`, the MQT compiler package reports
`11 passed`, and the QFw MQT helper reports `4 passed`.

The opt-in managed-QFw gate also ran inside an initialized DEFw Slurm step as
job 50. It reported `1 passed, 4 warnings in 12.87s` and completed with exit
code 0. The gate exercises allocation discovery, route binding, compilation,
submission, operation identity, status, cancellation, correlated terminal
delivery, listener close, and finalization against NWQSim.

## Cleanup and retained evidence

Stop only the site service started for this gate:

```bash
docker exec \
  -e QFW_INSTALL_PREFIX=/workspace/qfw-container-base/qfw-main-install \
  -e QFW_VENV=/workspace/qfw-container-base/qfw-main-venv \
  slurmctld \
  /workspace/qfw-container-base/QFw-release-v0.1/repos-main-sync/\
QFw-SLURM-Cluster/tools/qfw-site-services stop --target nwqsim
```

Retain the two job output files and their per-backend child logs. They contain
package versions, class identities, routes, operation IDs, compiler metadata,
provenance, and results. They contain no credentials, provider secrets, or
serialized remote service objects. Jobs 21 through 36, 38 through 40, and 42
through 49 were diagnostic runs and are not qualification evidence.
