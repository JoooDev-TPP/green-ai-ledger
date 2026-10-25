<img src=".github/banner.svg" width="100%" alt="green-ai-ledger: compute carbon with the grid factor pinned to a cited source">

[![tests](https://github.com/JoooDev-TPP/green-ai-ledger/actions/workflows/test.yml/badge.svg)](https://github.com/JoooDev-TPP/green-ai-ledger/actions/workflows/test.yml) [![License: MIT](https://img.shields.io/github/license/JoooDev-TPP/green-ai-ledger)](LICENSE)

Measure what your compute costs in carbon. The grid factor is pinned to a cited official source. It is never guessed from your IP address.

## Quick start

```bash
git clone https://github.com/JoooDev-TPP/green-ai-ledger
cd green-ai-ledger
pip install -e ".[dev]"

ledger factors
ledger run --label "test suite" --project demo -- make test
ledger report --out reports/green-ai.md
```

`ledger factors` prints the loaded grid factors and their citation. `ledger run` runs any command after `--`, records its energy and CO2e, and passes the exit code through. `ledger report` writes the markdown ledger. Python 3.11 or later; this installs codecarbon and typer.

## The IP problem

The same 2.29 Wh job, scored with the factor a tracker would apply depending on where the machine's public IP exits (CodeCarbon 3.3.1 factors):

| Public IP resolves to | Factor applied | Result | Error |
|---|---|---|---|
| **Guangdong grid (correct)** | **0.4419** | **1.01 g** | |
| Hong Kong | 0.6995 | 1.60 g | +58% |
| China, national average | 0.5823 | 1.33 g | +32% |
| Japan | 0.4854 | 1.11 g | +10% |
| South Korea | 0.4306 | 0.98 g | −3% |
| United States | 0.3695 | 0.84 g | −16% |
| Iceland | 0.0277 | 0.06 g | −94% |

Same machine, same work. The answer ranges from half again too much to a sixteenth of the right one. Even the right country is 32% off, because a national average is not a province.

A carbon tracker run out of the box geolocates the machine from its public IP and applies that country's grid factor. A public IP shows where the traffic leaves. Behind a corporate proxy, a cloud relay, a VPN or a university gateway, the tracker applies the grid of the exit. The number looks plausible, carries a country name, and nobody re-checks it.

So this tool does not geolocate. The region is an argument. An unknown region is an error, with no fallback. Every factor carries its issuing body, document, URL, publication date and the SHA-256 of the source file.

```
$ ledger factors
  全国     0.5306   kgCO2/kWh  [national]
  南方     0.4042   kgCO2/kWh  [regional]
  广东     0.4419   kgCO2/kWh  [provincial]
  四川     0.1564   kgCO2/kWh  [provincial]
  ...
```

Below the table it prints the citation, 生态环境部、国家统计局，关于发布2023年电力二氧化碳排放因子的公告，2025-12-31 (the Ministry of Ecology and Environment and National Bureau of Statistics notice on the 2023 electricity CO2 emission factors), and the SHA-256 of the source file, `d43b60a3…a3f018e5a`. Within one country the factor runs from 0.1564 in Sichuan to 0.6361 in the north, a factor of four.

## Measured or estimated

macOS exposes real per-subsystem power through `powermetrics`, which needs root. Without root, every tool, this one included, falls back to the processor's nameplate wattage multiplied by elapsed time. On a laptop that idles between bursts, that can be out by several times.

Every record therefore carries a `method` field. The report totals measured and estimated records separately and never adds across them. A ledger made only of estimates says so in its first sentence:

> **全部记录为估算，不是实测。** 本机没有以 root 运行 `powermetrics`，因此功率取的是处理器铭牌值乘以时长。笔记本在任务之间会闲置，这种估算可能偏高数倍。
>
> *(All records are estimates, not measurements: `powermetrics` was not run as root, so power is nameplate × duration, which can overstate a laptop that idles between tasks by several times.)*

Raw kilowatt-hours are stored alongside the factor, so either can be corrected later without re-running the work. The recorded command line writes the home directory as `~`, because the ledger is meant to be committed next to the work it measures.

## What it measured

Three runs across two repositories, on an M1 Pro laptop, Guangdong grid ([full ledger](reports/green-ai.md)):

| Project | Task | Duration | Energy | CO2e |
|---|---|---|---|---|
| pv-wind-power-forecast | ultra-short-term training, 14 stations × 16 horizons | 134 s | 2.29 Wh | 1.01 g |
| [aiwp-china-verification](https://github.com/JoooDev-TPP/aiwp-china-verification) | verification over 198k forecast pairs | 4 s | 0.03 Wh | 0.01 g |
| aiwp-china-verification | figures | 4 s | 0.03 Wh | 0.01 g |

The heaviest job emitted one gram of CO2e, roughly what boiling a thimble of water costs. It is published anyway. A figure only means something if it gets reported when it is small as well as when it is large.

## Usage

```bash
# measure a command and record it
ledger run --label "…" --project repo -- <any command>
# use a different grid
ledger run --region 四川 --label "…" --project repo -- …
# write the markdown ledger
ledger report --out reports/green-ai.md
# one line for a README
ledger badge --project repo
```

Because `ledger run` passes the command's exit code through, it drops into a Makefile without changing what a failure means.

## Scope

Local compute only: the CPU, GPU and memory of the machine the command runs on, for as long as it runs. It leaves out the manufacture of the hardware, the network, and anything the command calls out to. It is not a life-cycle assessment.

## Layout

```
src/ledger/
  grid.py      the factors, their citation, the refusal to geolocate
  measure.py   run something, record energy and how it was measured
  report.py    markdown ledger, measured and estimated kept apart
  cli.py       ledger run / report / badge / factors
tests/         pytest
```

## License

MIT. For the grid factors, see the citation printed by `ledger factors`.

See also [Carbon-Footprint-LCA](https://github.com/JoooDev-TPP/Carbon-Footprint-LCA), which applies the same 2023 grid factors to lifecycle emissions of electricity.
