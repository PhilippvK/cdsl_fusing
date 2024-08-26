# CoreDSL Fusing Experiments

## Setup

TODO

## Instructions

### Parse original CDSL
```
python3 -m m2isar.frontends.coredsl2_set.parser inputs/example_flat.core_desc -I ./etiss_arch_riscv/rv_base
python3 -m m2isar.frontends.coredsl2_set.parser inputs/example_flat_manual.core_desc -I ./etiss_arch_riscv/rv_base
```

### Add encoding to unencoded instructions
```
python3 -m m2isar.transforms.encode_instructions.encoder gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.encode_instructions.encoder gen_model/example_flat_manual.m2isarmodel --inplace
```


### Fusing procedure (flat)
```
python3 -m m2isar.transforms.collect_outline_tasks.collect gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.drop_unused.optimizer gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.eliminate_mod_rfs.transform gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.optimize_instructions.optimizer gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.eliminate_rd_cmp_zero.transform gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.optimize_instructions.optimizer gen_model/example_flat.m2isarmodel --inplace
# TODO: remove raise
python3 -m m2isar.transforms.eliminate_scalar_assignments.transform gen_model/example_flat.m2isarmodel --inplace
# TODO: split I/O
# TODO: assert no control flow (optional)
# TODO: outline behavior
python3 -m m2isar.transforms.inline_functions.transform gen_model/example_flat.m2isarmodel --inplace
python3 -m m2isar.transforms.drop_unused.optimizer gen_model/example_flat.m2isarmodel --inplace
# TODO: add mod rfs (optional)
# TODO: add rd != 0 check (optional)
# TODO: add raises? (optional)
```

### Fusing procedure (flat_manual)
```
python3 -m m2isar.transforms.drop_others.optimizer gen_model/example_flat_manual.m2isarmodel --inplace
python3 -m m2isar.transforms.drop_unused.optimizer gen_model/example_flat_manual.m2isarmodel --inplace
python3 -m m2isar.transforms.inline_functions.transform gen_model/example_flat_manual.m2isarmodel --inplace
python3 -m m2isar.transforms.eliminate_scalar_assignments.transform gen_model/example_flat.m2isarmodel --inplace
# TODO: add mod rfs (optional)
# TODO: add rd != 0 check (optional)
# TODO: add raises? (optional)
```

### Write resulting CDSL
```
python -m m2isar.backends.coredsl2_set.writer ./gen_model/example_flat.m2isarmodel -o outputs/example_flat_out.core_desc
python -m m2isar.backends.coredsl2_set.writer ./gen_model/example_flat_manual.m2isarmodel -o outputs/example_flat_manual_out.core_desc
```
