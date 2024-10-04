# CoreDSL Fusing Experiments

## Setup

```
git submodule update --init --recursive
virtualenv -p python3 venv
source venv/bin/activate
pip install -e M2-ISA-R
PYTHONPATH=$(pwd)/M2-ISA-R:$PYTHONPATH
```

## Instructions

### Parse original CDSL
```
python3 -m m2isar.frontends.coredsl2_set.parser inputs/example_flat.core_desc -I ./etiss_arch_riscv/rv_base
# python3 -m m2isar.frontends.coredsl2_set.parser inputs/example_flat_manual.core_desc -I ./etiss_arch_riscv/rv_base
```

### Add encoding to unencoded instructions
```
python3 -m m2isar.transforms.encode_instructions.encoder inputs/gen_model/example_flat.m2isarmodel --inplace
# python3 -m m2isar.transforms.encode_instructions.encoder inputs/gen_model/example_flat_manual.m2isarmodel --inplace
```


### Fusing procedure (flat)
```
python3 -m m2isar.transforms.collect_outline_tasks.collect inputs/gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.drop_unused.optimizer inputs/gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.eliminate_mod_rfs.transform inputs/gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.optimize_instructions.optimizer inputs/gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.eliminate_rd_cmp_zero.transform inputs/gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.optimize_instructions.optimizer inputs/gen_model/example_flat.m2isarmodel --inplace
# TODO: remove raise
python3 -m m2isar.transforms.eliminate_scalar_assignments.transform inputs/gen_model/example_flat.m2isarmodel --inplace
# TODO: split I/O
# TODO: assert no control flow (optional)
# TODO: outline behavior
python3 -m m2isar.transforms.inline_functions.transform inputs/gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.drop_unused.optimizer inputs/gen_model/example_flat.m2isarmodel --inplace
# TODO: add mod rfs (optional)
# TODO: add rd != 0 check (optional)
# TODO: add raises? (optional)
```

### Fusing procedure (flat_manual)
```
python3 -m m2isar.transforms.drop_others.optimizer inputs/gen_model/example_flat_manual.m2isarmodel --inplace
python3 -m m2isar.transforms.drop_unused.optimizer inputs/gen_model/example_flat_manual.m2isarmodel --inplace
python3 -m m2isar.transforms.inline_functions.transform inputs/gen_model/example_flat_manual.m2isarmodel --inplace
python3 -m m2isar.transforms.eliminate_scalar_assignments.transform inputs/gen_model/example_flat.m2isarmodel --inplace
# TODO: add mod rfs (optional)
# TODO: add rd != 0 check (optional)
# TODO: add raises? (optional)
```

### Write resulting CDSL
```
python -m m2isar.backends.coredsl2_set.writer inputs/gen_model/example_flat.m2isarmodel -o outputs/example_flat_out.core_desc --write-operands
# python -m m2isar.backends.coredsl2_set.writer inputs/gen_model/example_flat_manual.m2isarmodel -o outputs/example_flat_manual_out.core_desc --write-operands
```

## Workarounds

Parser seems to have issues if one file is imported multiple times. Comment out all but the first include and it should work... (i.e. in `RVM.core_desc`)
