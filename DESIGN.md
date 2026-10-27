# Decisions and tradeoffs

Update as I go

Created layout that keeps estimating logic testable and separate from game layer.
```
blockbid/
├── README.md                 # problem, scope, demo GIF, quickstart
├── DESIGN.md                 # decisions and tradeoffs, updated as you go
├── LICENSE
├── pyproject.toml            # deps, ruff/pytest/mypy config
├── .github/
│   └── workflows/ci.yml      # lint + type check + tests on every push
├── src/
│   └── blockbid/
│       ├── __init__.py
│       ├── core/             # pure Python, no game imports
│       │   ├── models.py     # Material, Trade, LineItem, Estimate (dataclasses)
│       │   ├── pricing.py    # unit prices, regional/markup adjustments
│       │   ├── labor.py      # time/labor estimates per trade item
│       │   └── estimator.py  # blocks in -> Estimate out
│       ├── mapping/
│       │   ├── block_map.py  # game block -> material/trade item
│       │   └── scales.py     # 1:1 vs miniature scale conversion
│       ├── adapters/         # game-specific code lives only here
│       │   ├── base.py       # abstract WorldReader interface
│       │   └── minecraft.py  # reads a build/schematic into core models
│       ├── export/
│       │   ├── csv_export.py
│       │   └── report.py     # human-readable estimate summary
│       └── cli.py            # `blockbid estimate path/to/build`
├── data/
│   ├── prices.json           # sample unit costs
│   └── block_map.json        # editable mapping, no code changes needed
├── tests/
│   ├── core/                 # estimator, pricing, labor unit tests
│   ├── mapping/              # scale conversion edge cases
│   ├── adapters/             # run against fixture builds
│   └── fixtures/             # tiny sample builds (e.g. one room)
├── examples/
│   └── single_room/          # sample input plus expected estimate
└── docs/
    └── architecture.md       # one diagram: adapter -> mapping -> core -> export
```
Reasoning:
core/ never imports game code. The estimator takes a plain list of block counts and returns an Estimate, to test with ordinary pytest and no Minecraft running. Reviewers check this first.
adapters/ is the only place that knows about a specific game. When adding Fortnite or browser version later, just add one new adapter and core/ doesnt change.
Mappings and prices live in data/ as JSON. Changing what a block represents (example, a block as 1 foot of conduit) is a data edit, not a code edit, and the mapping is easy to test.
scales.py is separate because 1:1 vs miniature option is a classic source of off-by-one and unit bugs. Give it its own thorough tests.
tests/fixtures/ and examples/ give a reviewer a one-command demo that proves the project works end to end.
