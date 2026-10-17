# FAC-CR-NGS

FAC-CR-NGS aims to screen biosynthetic gene clusters associated with secondary metabolism in large FAC libraries.

**Status: design stage.** At the audited commit, this repository contains a project description and GPLv3 license, but no executable analysis pipeline or validation dataset. Assembly and candidate classification designs are informed by a static review of FAC-assembly; they have not been validated for this project. No speed, accuracy, or experimental outcome is claimed.

## Documentation

- [Repository evidence and reuse audit](docs/repository-audit.md)
- [Proposed workflow and data contracts](docs/workflow-spec.md)
- [File-level implementation plan and acceptance criteria](docs/implementation-plan.md)
- [Header-only FAC end-sequence template](examples/fac_ends.header.tsv)

The meaning of CR, pool design, sample manifest, vector identity, subtraction references, and BGC screening criteria remain to be confirmed. The header template is not experimental data.

## Provenance and licensing

The reviewed source is [hychen86/FAC-assembly](https://github.com/hychen86/FAC-assembly/tree/12fc8d225b7e9f8e1a9090a6272294467b2f014d), a fork of nextgenusfs/FAC-assembly, with source scripts credited to Jon Palmer (2018). Its BSD 2-Clause license requires retention of the copyright notice, conditions, and disclaimer when redistributing source. This documentation proposal includes no migrated source code. Any future copied code or documentation must retain its source attribution and license notices. The existing FAC-CR-NGS LICENSE is retained.
