# Local ports

The host ports each repo's `docker compose` binds on a developer's machine, so every
component can run at once without colliding with each other or with what is commonly
already running (5432, 8000, 8080, 3000, 5000 and Supabase's 54321-54329 among them). See
L-0014.

Each repo gets a block of ten in 47100-47199: the block's `0` is its main service, its `9`
its database, and the numbers between are anything else it runs. Inside the Compose
network, services keep their usual container ports (Postgres on 5432 and so on); only the
host side is fixed here. A repo's Compose file reads each port from an environment
variable, defaulting to this table, so one machine can still move one.

| Repo | Block | Port | Service |
| --- | --- | --- | --- |
| `store` | 47110 | 47110 | API |
| | | 47119 | PostGIS |
| `georef` | 47120 | 47120 | Allmaps editor |
| | | 47121 | IIIF image server |
| | | 47122 | Allmaps Live API |
| | | 47123 | Allmaps REST API |
| | | 47124 | Allmaps Annotations API |
| | | 47125 | Allmaps Tile Server |
| | | 47126 | Allmaps Preview |
| | | 47127 | Allmaps Viewer |
| | | 47129 | Postgres |
| `cite` | 47130 | 47130 | cite server (Ask AI and background requests) |
| `corpus` | 47140 | 47140 | search API |
| | | 47149 | index database |
| `trace` | 47150 | 47150 | reserved |
| `review` | 47160 | 47160 | web app |
| `release` | 47170 | 47170 | reserved |
| `backport` | 47180 | 47180 | reserved (imports are jobs and bind nothing) |

A new repo takes the next free block. A new service takes the next free number in its
repo's block, and gets a row here in the same PR that adds it to Compose.
