# Glossary

Acronyms stay in English with their meaning. Entries marked "(general knowledge, to verify in phase 4)" come from general telecom knowledge and have not yet been checked against 3GPP TS 23.040 / TS 29.002. Entries marked "(Jasmin docs)" were checked in the Jasmin documentation. Entries marked "(observed)" were seen in this lab.

## SMS and mobile network

| Acronym | Meaning | In this lab |
|---|---|---|
| SMS | Short Message Service | The service the lab is about |
| SMSC | Short Message Service Center | Server that accepts, keeps and delivers SMS |
| MO | Mobile Originated | SMS sent from a handset |
| MT | Mobile Terminated | SMS delivered to a handset |
| A2P | Application-to-Person | SMS sent by an application to a person |
| P2P | Person-to-Person | SMS between people |
| SS7 | Signaling System No. 7 | Telecom signalling stack that MAP runs on |
| MAP | Mobile Application Part | SS7 protocol used between mobile-network nodes |
| HLR | Home Location Register | Subscriber database |
| MSC | Mobile Switching Center | Switch serving the handset |
| VLR | Visitor Location Register | Database of subscribers served by an MSC |
| MO-ForwardSM | Mobile Originated Forward Short Message | MAP operation, MSC to SMSC (general knowledge, to verify in phase 4) |
| SRI-for-SM | Send Routing Info for Short Message | MAP query, SMSC to HLR (general knowledge, to verify in phase 4) |
| MT-ForwardSM | Mobile Terminated Forward Short Message | MAP operation, SMSC to serving MSC (general knowledge, to verify in phase 4) |

## SMPP

| Term | Meaning | In this lab |
|---|---|---|
| SMPP | Short Message Peer-to-Peer | TCP protocol between an ESME and an SMSC |
| ESME | External Short Messaging Entity | An application that connects to an SMSC over SMPP |
| PDU | Protocol Data Unit | One SMPP message on the wire |
| bind | SMPP login | First call of a session: transmitter, receiver or transceiver |
| BIND_TRANSCEIVER | Bind as transceiver | Session that can send and receive (observed: count 1 on the SMPPSim panel) |
| BOUND_TRX | Session state shown by jCli | The connector is bound as transceiver (observed) |
| submit_sm | Submit Short Message | PDU an ESME sends to submit an SMS |
| deliver_sm | Deliver Short Message | PDU from the SMSC to the ESME (incoming SMS or delivery receipt) |
| DLR | Delivery Receipt | Report on what happened to a message |
| ENQUIRE_LINK | Keep-alive PDU | Seen counting on the SMPPSim panel (observed) |
| ESME_ROK | Status "no error" | Column on the SMPPSim panel counting successful PDUs (observed) |

## Jasmin

| Term | Meaning | In this lab |
|---|---|---|
| jCli | Jasmin command-line console | Management console on port 8990 |
| smppccm | SMPP Client connector manager | jCli command for outgoing SMPP connectors (Jasmin docs) |
| httpccm | HTTP Client connector manager | jCli command for HTTP connectors (Jasmin docs); not used in the lab |
| connector | Outgoing connection from Jasmin to an upstream | Here: `smppsim` |
| MT route | Rule choosing the connector for a message to deliver | Here: `DefaultRoute`, order 0 |
| DefaultRoute | Route type that matches everything | Sends to `smppc(smppsim)` |
| persist | jCli command that saves configuration to disk | Writes profile `jcli-prod` in `/etc/jasmin/store` |
| ND | No data | Shown by jCli when a value is not set; `(!)` is an attention marker, not an error |
| store&forward (Jasmin) | Messages wait in queues inside the gateway | RabbitMQ-based; not the same as SMSC store-and-forward |
| AMQP | Advanced Message Queuing Protocol | Protocol RabbitMQ speaks |

## Docker

| Term | Meaning |
|---|---|
| YAML | Text format of Compose files (YAML Ain't Markup Language) |
| Named volume | Docker-managed storage with a name; the lab uses `jasmin-store` |
| Anonymous volume | Volume created without a name (for example from an image's `VOLUME` line) |
| Dangling volume | Volume no container uses any more; list with `docker volume ls --filter dangling=true` |
