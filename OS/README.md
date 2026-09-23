**Distributed Key-Value Store with Raft Consensus and LSM-Tree Storage**

Trimester Project --- Live Build | Operating Systems / Distributed
Systems

A production-oriented distributed key-value store built from scratch in
Go, using Raft consensus for replicated state, an LSM-tree
storage engine for persistent data, gRPC for client communication,
and Kubernetes for multi-node deployment and observability. The
system is demonstrated through a Library Management use case.

**Project Status**

Status: In Progress --- Live Build

The project is developed incrementally: basic KV store → WAL/MemTable →
LSM-tree → concurrency → gRPC → Raft → Kubernetes → observability →
fault injection → benchmarking.

**Objectives**

Build a distributed key-value store from scratch.

Implement persistent LSM-tree storage.

Implement Raft consensus and replicated state.

Provide gRPC client communication.

Support safe concurrent requests and pass the Go race detector.

Deploy a multi-node cluster with Kubernetes and Helm.

Add Prometheus metrics and Grafana dashboards.

Perform Jepsen-style fault-injection experiments.

Measure throughput, latency, recovery time, CPU, memory and storage
behavior.

Compare measured performance with established systems such as
RocksDB, etcd and TiKV where practical.

**Architecture**

Library Client
      |
     gRPC
      v
Client Router
      |
      v
Raft Leader
   /       \
Node 2    Node 3
   \       /
    Replicated Log
          |
          v
   WAL + MemTable
          |
       SSTables
          |
      Compaction
          |
    Persistent Disk

Kubernetes hosts the nodes; Prometheus collects metrics and Grafana visualizes them.

**Library Management Use Case**

Example records:

Key                Value

book:101         {"title":"Operating Systems","available":true}
book:102         {"title":"Distributed Systems","available":false}
member:501       {"name":"Student 501"}
borrow:501:101   {"status":"borrowed"}

Core operations: PUT, GET, DELETE. The library application is the
workload used to demonstrate the underlying distributed-systems and OS
concepts.

**LSM-Tree Storage**

Client request → WAL → MemTable → flush → SSTable → compaction

WAL: records operations before durable processing and supports
recovery.

MemTable: in-memory ordered recent state.

SSTable: immutable sorted on-disk table produced by flushing.

Tombstone: represents deletes.

Compaction: merges SSTables, removes obsolete
versions/tombstones when safe, and controls read/storage overhead.

**Raft Consensus**

A typical three-node cluster has one leader and followers. A write is
appended to the leader's log, replicated to followers, committed after
the required majority, and applied to the state machine. If the leader
fails, another node can be elected.

Important components: - leader election - terms and election timeouts -
heartbeats - log replication - commit index - state-machine
application - persistent Raft state - safety tests

gRPC and At-Least-Once Semantics

The client communicates with the store through Protocol Buffers and
gRPC. The service will expose operations such as Put, Get, and
Delete. Client routing will direct requests to the current leader.
Retries may cause duplicate delivery, so request/operation IDs and
idempotency rules will be used where needed for library actions.

**Concurrency**

The store must support multiple simultaneous clients. Go goroutines,
mutexes/RWMutexes, channels and atomic operations will be used where
appropriate. Validation includes:

go test ./...
go test -race ./...
go test -bench=. ./...

**Kubernetes and Helm**

The final deployment will run multiple KV nodes in Kubernetes. A Helm
chart will package the deployment and services. Local development can
use kind; the deployment can be adapted to managed Kubernetes such
as EKS/GKE.

**Observability**

Prometheus will collect metrics such as request count, errors, request
latency, Raft leader changes, commit progress, flushes and compactions.
Grafana will display dashboards for request rate, latency, resource
usage, leader status and errors. Dashboard JSON will be versioned in
GitHub.

**Fault Injection**

Controlled failures will include leader termination, follower
termination, restart/rejoin and other documented disruptions. Each
experiment will record the injected fault, expected behavior, observed
behavior, recovery time, consistency result, logs and metrics.

Example:

Node 1 = Leader
Node 2 = Follower
Node 3 = Follower
        ↓
Kill Node 1
        ↓
Node 2 becomes Leader
        ↓
Writes continue through the new leader

**Weekly Milestones**

**Weeks 1--2 --- Foundation**

Topic locked

Linux profiling baseline

GitHub repository and Go scaffold

basic KV API

**Weeks 3--4 --- LSM Storage**

WAL

MemTable

SSTables

reads/writes/deletes

compaction

recovery

benchmark against RocksDB where practical

**Weeks 5--6 --- Concurrency**

concurrent in-memory structures

race-detector validation

benchmarks

**Weeks 7--8 --- Distributed Primitives**

gRPC service

client routing

multi-node communication

at-least-once semantics

**Weeks 9--10 --- Raft**

leader election

heartbeats

log replication

commit logic

state-machine application

safety tests

**Weeks 11--12 --- Production Infrastructure**

Kubernetes deployment

Helm chart

Prometheus

Grafana

Jepsen-style fault injection

final benchmarks and live demo

Deliverables

Distributed KV store source code in Go

LSM storage library

Raft implementation with safety tests

Helm chart

Prometheus configuration

Grafana dashboard JSON

Jepsen-style test harness and documented results

10--15 page ADR-style architecture document

multi-node Kubernetes live demo

performance benchmark report versus selected reference systems

**Repository Structure**

distributed-kv-store/
├── cmd/
│   ├── kvnode/
│   └── client/
├── internal/
│   ├── api/
│   ├── raft/
│   ├── lsm/
│   ├── wal/
│   ├── grpc/
│   ├── cluster/
│   └── metrics/
├── proto/
│   └── kv.proto
├── tests/
│   ├── integration/
│   ├── fault-injection/
│   └── consistency/
├── bench/
├── helm/
│   └── kvstore/
├── grafana/
├── prometheus/
├── docs/
│   ├── architecture.md
│   └── adr/
├── scripts/
├── go.mod
├── go.sum
└── README.md

**Technology Stack**

Area                Technology

Language            Go
Consensus           Raft
Storage             LSM-tree, WAL, SSTables
RPC                 gRPC + Protocol Buffers
Containers          Docker
Orchestration       Kubernetes
Packaging           Helm
Metrics             Prometheus
Dashboards          Grafana
Local cluster       kind
Testing             Go testing + race detector
Benchmarking        Go benchmarks
Version control     Git + GitHub
Reference systems   RocksDB / etcd / TiKV

**Development and Run Commands**

Prerequisites: Go, Git, Docker, kubectl, kind, Helm and protoc/gRPC
plugins.

go version
git --version
docker --version
kubectl version --client
kind version
helm version

**Run tests:**

go test ./...
go test -race ./...

Run the current development node:

go run ./cmd/kvnode -addr :8080 -data ./data

**Example health check:**

curl http://localhost:8080/health
# ok

**Example KV operations:**

curl -X PUT http://localhost:8080/kv/book:101 \
  -H "Content-Type: application/json" \
  -d '{"value":"Operating Systems"}'

curl http://localhost:8080/kv/book:101

curl -X DELETE http://localhost:8080/kv/book:101

The exact commands and endpoints should be updated if the final gRPC
interface differs from the initial development API.

**Benchmarking**

The final report will record throughput, average/p50/p95/p99 latency,
CPU, memory, storage usage, compaction cost and recovery time. Every
reported number should include the workload, node count,
hardware/environment and command/configuration used.

**Testing Evidence**

The repository should contain evidence for:

unit tests

race-detector output

LSM recovery tests

Raft election tests

replication tests

integration tests

leader/follower failure tests

node restart/rejoin tests

**benchmark output**

Kubernetes deployment

Prometheus/Grafana screenshots

fault-injection results

**Final Demo**

Start a three-node cluster.

Show the current leader.

Write a library record.

Read it back.

Show replication.

Stop the leader.

Demonstrate new leader election.

Continue reads/writes.

Restart the failed node and show recovery/catch-up.

Show Grafana metrics.

Present measured benchmark and fault-test results.

**Team Contributions**

Member 1: LSM/WAL/storage and storage benchmarks

Member 2: Raft/consensus/fault injection

Member 3: gRPC/Kubernetes/Helm/Prometheus/Grafana

All team members should understand the complete architecture for the
final live demonstration.

Architecture Decisions

The docs/adr/ directory will record decisions such as:

ADR-001: Why Go?

ADR-002: Why an LSM-tree?

ADR-003: Why Raft?

ADR-004: Why gRPC?

ADR-005: Why Kubernetes?

Research / Engineering Questions

How does LSM-tree storage behave under write-heavy workloads?

How does compaction affect latency?

What is the overhead of replication?

How quickly does the cluster recover after leader failure?

How does latency change with concurrent clients?

How does the implementation compare with reference systems under the
same workload?

Limitations

This is an educational systems project and should not automatically be
considered a production database. The final report will document the
implemented scope, assumptions, limitations, security model, supported
cluster size, and benchmark environment.

Future Enhancements

TLS/mTLS

authentication and authorization

snapshots

Bloom filters and SSTable indexes

advanced compaction policies

dynamic membership

stronger client libraries

cloud deployment automation

larger-scale benchmarking

Team

Project: Distributed Key-Value Store with Raft Consensus and
LSM-Tree Storage

Application: Library Management

Program: M.Tech Computer Science Engineering

Team Members: - Member 1 --- <Name> - Member 2 --- <Name> -
Member 3 --- <Name>

Institution: <College / University>

Summary

This project implements a distributed key-value storage system from
scratch. It combines LSM-tree persistent storage, Raft consensus, gRPC
communication, concurrent request handling, Kubernetes deployment,
Prometheus/Grafana observability, and controlled fault-injection
testing. The Library Management workload demonstrates operations such as
adding books, retrieving book information, tracking members, and
recording borrowing/return events. The project is developed as a live
build: each subsystem is implemented, tested, measured and documented
before the next layer is added.
