# Amendment 02 Phase 7 Compiler Package Integration

This gate composes the two compiler packages in separate Python 3.12
environments. The split matches QFw-SLURM-Cluster and avoids claiming that
Qiskit 2.2.3 and the MQT-qualified Qiskit 2.5.2 can coexist.

## Frozen revisions

- libquantum `06fb07f`, version `0.1.0.dev0`
- QFw `0de938d`, helper distribution `qfw-qiskit==0.0.1`
- libquantum-compiler-qiskit `6334f8f`, version `0.1.0`
- libquantum-compiler-mqt `95cbe08`, version `0.1.0`
- qfw-slurm `e940e0a60022`
- QFw-SLURM-Cluster `9934f3fbcca8`
- Qiskit environment `qiskit==2.2.3`
- MQT environment `mqt-core==4.1.1`, `qiskit==2.5.2`

## Build and isolated installation

Set `QISKIT_PYTHON` and `MQT_PYTHON` to interpreters containing the exact SDK
versions above. From any directory, run:

```bash
WHEELS=$(mktemp -d /tmp/libquantum-phase7-wheels-XXXXXX)
QISKIT_TARGET=$(mktemp -d /tmp/libquantum-phase7-qiskit-XXXXXX)
MQT_TARGET=$(mktemp -d /tmp/libquantum-phase7-mqt-XXXXXX)

"$QISKIT_PYTHON" -m pip wheel --no-deps -w "$WHEELS" \
  /path/to/libquantum \
  /path/to/QFw/backends \
  /path/to/libquantum-compiler-qiskit \
  /path/to/libquantum-compiler-mqt

"$QISKIT_PYTHON" -m pip install --no-deps \
  --target "$QISKIT_TARGET" \
  "$WHEELS"/libquantum-0.1.0.dev0-*.whl \
  "$WHEELS"/qfw_qiskit-0.0.1-*.whl \
  "$WHEELS"/libquantum_compiler_qiskit-0.1.0-*.whl

"$MQT_PYTHON" -m pip install --no-deps \
  --target "$MQT_TARGET" \
  "$WHEELS"/libquantum-0.1.0.dev0-*.whl \
  "$WHEELS"/qfw_qiskit-0.0.1-*.whl \
  "$WHEELS"/libquantum_compiler_mqt-0.1.0-*.whl
```

Run each repository's tests from `/tmp` so its source package cannot shadow
the installed wheel. The extra libquantum path supplies only its reusable test
fixture package to the Qiskit test suite.

```bash
cd /tmp
PYTHONPATH="$QISKIT_TARGET:/path/to/libquantum" \
  "$QISKIT_PYTHON" -m pytest -q \
  /path/to/libquantum-compiler-qiskit/tests
PYTHONPATH="$MQT_TARGET" "$MQT_PYTHON" -m pytest -q \
  /path/to/libquantum-compiler-mqt/tests
```

Both commands must report `10 passed`.

## Retained integration trace

Both package gates use the same `dummy:compiler` qhw fixture with nodes
`QB1`, `QB2`, a directed `QB1 -> QB2` edge, operations `rz`, `sx`, `x`, `cx`,
and `measure`, and no calibration record. Each QPM advertises OpenQASM 2.0.

Each packaged `compiler.json` is discovered through `plugin_path()` by the
normal libquantum loader. Each environment exposes one compiler route. Route
listing leaves the target cache empty and does not invoke a compiler callback.
Compilation passes the three qhw objects to the QFw helper by identity and
leaves the source and qhw records unchanged.

The Qiskit trace contains `qiskit_transpile`, `qasm2_basis_lowering`, and
`qasm2_serialize`. The MQT converted-input trace also contains
`qiskit_to_mqt`, followed by target compilation and the qualified OpenQASM 3
to OpenQASM 2 conversion stages. Both outputs round-trip through the exact
environment's Qiskit OpenQASM 2 parser.

Before explicit submission, each dummy QPM submission list is empty. The test
then calls `Context.run()` and asserts that QRI receives the exact compiler
payload object by identity. It marks the operation complete and finalizes the
context, which unregisters and closes the completion endpoint.

## Core-only import

Install only the libquantum wheel into another empty target and import it.
The import must not add Qiskit, MQT, QFw, or either compiler package to
`sys.modules`.

```bash
CORE_TARGET=$(mktemp -d /tmp/libquantum-phase7-core-XXXXXX)
"$QISKIT_PYTHON" -m pip install --no-deps \
  --target "$CORE_TARGET" \
  "$WHEELS"/libquantum-0.1.0.dev0-*.whl
cd /tmp
PYTHONPATH="$CORE_TARGET" "$QISKIT_PYTHON" -c \
  'import libquantum; print(libquantum.__file__)'
```

The retained run reported `core-only import clean`. Temporary wheel and target
directories contain no credentials and may be removed after the evidence is
recorded.
