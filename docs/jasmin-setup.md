# Jasmin setup: configuring the gateway with jCli

This page records, step by step, how the lab gateway was configured. The commands and the outputs shown in the code blocks were typed and observed in this lab (Jasmin 0.11.0, SMPPSim 2.6.3, 2026-10-18). The text around them is explanation: it comes from the Jasmin documentation or from general Docker and SMPP knowledge, and it was not tested separately here.

> The credentials in this page (`jcliadmin`/`jclipwd`, `lab_user`/`lab_pass`, `smppclient1`/`password`) are defaults or throwaway values for a lab whose ports are published only on `127.0.0.1`. Do not reuse them anywhere else.

## The pieces, in my own words

Jasmin is configured through **jCli**, its command-line console. The model has four layers:

| Piece | What it is | jCli command |
|---|---|---|
| Group | A container for users. | `group` |
| User | The account an ESME (a client application) uses to bind to Jasmin on port 2775. | `user` |
| SMPP client connector | An outgoing SMPP connection from Jasmin to an upstream SMSC. Here, the upstream is SMPPSim. | `smppccm` (SMPP Client connector manager) |
| MT route | A rule that says which connector carries a message Jasmin must deliver (MT = Mobile Terminated). | `mtrouter` |

`httpccm` (HTTP Client connector manager) is the HTTP equivalent of `smppccm`. The lab does not use it.

Sources for the names: the Jasmin documentation, https://docs.jasminsms.com, and the repository https://github.com/jookies/jasmin.

## 1. Open jCli

jCli listens on port 8990, published on `127.0.0.1`.

```bash
nc 127.0.0.1 8990
```

`nc` (netcat) opens a plain TCP connection. Jasmin asks for a username and a password (the defaults documented by Jasmin, `jcliadmin` / `jclipwd`, worked in this lab). The welcome banner reads `Welcome to Jasmin 0.11.0 console`.

Note: with `nc`, the terminal echoes what you type and jCli echoes it back too, so lines can appear twice.

## 2. Create a group

```
jcli : group -a
> gid lab
> ok
Successfully added Group [lab]
```

`-a` adds. At the `>` prompt you set fields; `ok` saves and `ko` leaves without saving. `group -l` lists groups (output: `#lab`, `Total Groups: 1`).

## 3. Create a user

```
jcli : user -a
> username lab_user
> password lab_pass
> gid lab
> uid lab_user
> ok
Successfully added User [lab_user] to Group [lab]
```

`user -l` then shows `lab_user`, group `lab`, and `ND (!)` under Balance and under MT SMS. `ND` means no data: those values are not set. `(!)` is only an attention marker (we checked this in Jasmin's source code), not an error.

## 4. Create the connector to SMPPSim

```
jcli : smppccm -a
> cid smppsim
> host smppsim
> username smppclient1
> password password
> bind transceiver
> ok
Successfully added connector [smppsim]
```

- `host smppsim` works because Docker Compose gives each service a DNS name equal to its service name.
- No port was set; the connector bound anyway, so the default port worked against SMPPSim in this lab.
- `bind transceiver` asks for a session that can both send and receive.

A new connector is `stopped`. Start it (`-1` starts, with the digit one) and list:

```
jcli : smppccm -1 smppsim
Successfully started connector id:smppsim
jcli : smppccm -l
#smppsim     started BOUND_TRX     1     0
```

`BOUND_TRX` means the SMPP session is bound as transceiver. SMPPSim's web panel at http://127.0.0.1:8089 also showed `TRX sessions bound: 1` and `BIND_TRANSCEIVER ESME_ROK count: 1`.

## 5. Create the MT route

```
jcli : mtrouter -a
> type DefaultRoute
> connector smppc(smppsim)
> rate 0.0
> ok
Successfully added MTRoute [DefaultRoute] with order:0
```

`mtrouter -l` shows `0 (!)` under Rate. Same meaning as before: the route is not rated (rate 0), not an error. A DefaultRoute with order 0 sends everything through `smppc(smppsim)`.

## 6. Save with persist

Everything above lives in memory until you run:

```
jcli : persist
group configuration persisted (profile:jcli-prod)
user configuration persisted (profile:jcli-prod)
...
```

`persist` writes the profile `jcli-prod` into `/etc/jasmin/store` inside the container. The Compose file mounts a named volume (`jasmin-store`) there, so the configuration survives container recreation. See [compose-explained.md](compose-explained.md#persistence-how-configuration-survives-restarts).

### Verified restart test

After `persist`, `docker compose down` followed by `docker compose up -d` recreated all containers. Without any jCli command, `smppccm -l` showed the connector `started` and `BOUND_TRX`, and group, user and route were still listed.

## What is not covered here

- Sending an SMS through the route: phase 2 (Python SMPP client).
- Stopping a connector, filters, interceptors and billing: not used in this lab, not verified here. See the Jasmin documentation.
