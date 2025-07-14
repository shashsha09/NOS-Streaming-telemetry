````markdown
# NOS Streaming Telemetry Lab

End-to-end demo of **Nokia SR OS** streaming telemetry on a single Ubuntu host  
(Stack: **containerlab → gNMIc → Prometheus → Grafana**).

| Component | Management IP | Default Creds | Notes |
|-----------|---------------|---------------|-------|
| **Telemetry Server** | **100.127.188.111** | `root / Nokia2018!` | Ubuntu 22/24 LTS; runs all containers |
| OTT-ESL-BNG-01 | 100.127.188.89 | `lab / Nokia_ESL!` | gNMI port 57400 |
| OTT-ESL-BNG-02 | 100.127.188.90 | `lab / Nokia_ESL!` | gNMI port 57400 |
| OTT-ESL-IXR-X01 | 100.127.188.162 | — | *(future expansion)* |
| OTT-ESL-IXR-X02 | 100.127.188.163 | — | *(future expansion)* |

---

## Quick Start

```bash
git clone https://github.com/shashsha09/NOS-Streaming-telemetry.git
cd NOS-Streaming-telemetry

# 1 – supply secrets (do NOT commit real passwords)
export GNMI_USER=lab
export GNMI_PASS=Nokia_ESL!
export GRAFANA_PASS=Nokia2018!

# 2 – deploy the full stack
containerlab deploy -t lab/clab.yml

# 3 – (optional) watch live telemetry scroll by
gnmic --config lab/configs/gnmic/gnmic-full-config.yml subscribe
````

| Service        | URL (after deploy)                                                                  |
| -------------- | ----------------------------------------------------------------------------------- |
| **Prometheus** | [http://100.127.188.111:9090](http://100.127.188.111:9090)                          |
| **Grafana**    | [http://100.127.188.111:3000](http://100.127.188.111:3000)  `admin / $GRAFANA_PASS` |

Destroy lab:

```bash
containerlab destroy -t lab/clab.yml
```

---

## Repository Layout

```
NOS-Streaming-telemetry/
├─ lab/
│  ├─ clab.yml                     # containerlab topology
│  └─ configs/
│     ├─ gnmic/                    # gNMIc subscriptions & processors
│     ├─ prometheus/               # prometheus.yml
│     └─ grafana/                  # datasource + dashboards
├─ scripts/                        # helper scripts (optional)
├─ docs/                           # diagrams / markdown (optional)
├─ Makefile                        # convenience targets (deploy / destroy)
└─ README.md
```

---

## How It Works 
1. **containerlab** starts 5 containers: `gnmic`, `prometheus`, `grafana`, `loki`, `promtail`.
2. **gNMIc** streams \~15 telemetry subscriptions (CPU, interface stats, DHCP, BGP…) from both BNGs.
3. **Prometheus** scrapes gNMIc at `http://localhost:9281/metrics` every 5 s.
4. **Grafana** auto-loads JSON dashboards from `lab/configs/grafana/dashboards/`.

---

## Handy CLI Snippets

```bash
# Check gNMI capabilities on BNG-01
gnmic -a 100.127.188.89:57400 -u $GNMI_USER -p $GNMI_PASS --insecure capabilities

# Live CPU feed (BNG-02)
gnmic -a 100.127.188.90:57400 -u $GNMI_USER -p $GNMI_PASS \
      --insecure subscribe \
      --path /state/system/cpu[sample-period=1]/summary/total/time-used

# Show containers started by containerlab
docker ps --filter name=telemetry-
```

---

## Troubleshooting FAQ

| Symptom                                        | Check / Fix                                                                                      |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Prometheus target **DOWN**                     | Verify `gnmic` container: `docker logs telemetry-gnmic`                                          |
| Grafana panel shows “No data”                  | Ensure dashboard variables `$device` / `$iface` have values and metric names begin with `gnmic_` |
| Prometheus warns **“server time out of sync”** | Run `sudo timedatectl set-ntp true` on 100.127.188.111                                           |
| Need extra metrics                             | Edit `lab/configs/gnmic/gnmic-full-config.yml`, add path, then restart `gnmic`                   |

---

## Contributing

1. Fork ➜ feature branch ➜ pull request.
2. GitHub Actions (CI) runs `containerlab deploy --dry-run` to catch YAML errors.
3. Add dashboards under `lab/configs/grafana/dashboards/`.

---

## License

MIT — see `LICENSE`.

*Maintainer*: [@shashsha09](https://github.com/shashsha09)  •  open an Issue for questions or improvements.

````

### How to use it

1. Save the block above as `README.md` in your repository root.  
2. Commit and push:

```bash
git add README.md
git commit -m "Add full lab README"
git push
````

Your GitHub repo will now display this comprehensive README for anyone who lands on the project.
