---
trigger: always_on
description: Communicate with industrial PLCs using Modbus TCP and EtherNet/IP (CIP) via github.com/iceisfun/goindustrial. Covers client/server setup, register and tag operations, reconnection, monitoring, protocol-agnostic PLC access, and optional GoLua scripting bindings.
---


# GoIndustrial Skill

Use this when helping someone who imported `github.com/iceisfun/goindustrial` and wants to communicate with Modbus TCP or EtherNet/IP devices.

## SKILLS

Copy-paste block for an AI assistant:

```text
SKILLS:
- GoIndustrial is a pure-Go library for Modbus TCP and EtherNet/IP (CIP) industrial protocols. Zero external dependencies.
- Two protocol clients, both implementing plc.PLC: modbus.Client and ethernetip.Client.
- Modbus quick path: modbus.Connect(ctx, host, opts...) -> client.ReadHoldingRegisters / WriteSingleRegister / etc.
- EtherNet/IP quick path: ethernetip.Connect(ctx, addr, opts...) -> client.ReadTag / WriteTag / ReadTagInto / etc.
- Both protocols support reconnecting transports: transport.NewReconnectingTransport[C](connector, closer, opts...).
- Functional options everywhere: modbus.WithRetries(3), modbus.WithUnitID(1), ethernetip.WithRetryDelay(2*time.Second), etc.
- Wire-level hex dump tracing: modbus.WithHexDump(os.Stdout) or ethernetip.WithHexDump(w) dumps all TCP traffic in hexdump -C format. Uses hexdump.Dumper which wraps io.Reader/io.Writer. Use io.MultiWriter to write to both stdout and a file.
- Monitor polls any plc.Reader: monitor.NewMonitor(reader) -> m.Subscribe(dataPoint, monitor.WithFrequency(100*ms)).
- Adaptive read clustering: monitor.NewClusteringReader(modbusClient, monitor.WithGapThreshold(32)) wraps a reader to coalesce nearby Modbus addresses into block reads. Reduces N subscriptions from N requests to ~1 per cluster. Supports cache TTL for cross-goroutine sharing, singleflight dedup, and WithClusteringEnabled(false) to force OFF.
- Clusterable interface: Modbus data point types (HoldingRegister, InputRegister, Coil, DiscreteInput) implement ClusterKey/ClusterAddr/ClusterQty/ClusterMerge/ClusterExtract for protocol-agnostic clustering.
- Modbus data areas: Coils (bool R/W), Discrete Inputs (bool R), Holding Registers (uint16 R/W), Input Registers (uint16 R).
- CIP data types: BOOL(0xC1), SINT(int8), INT(int16), DINT(int32), LINT(int64), USINT(uint8), UINT(uint16), UDINT(uint32), ULINT(uint64), REAL(float32), LREAL(float64), STRING(0xD0).
- Custom CIP types: implement cip.TypeCodec (MarshalCIP + UnmarshalCIP + CIPType()) and call cip.RegisterType(code, factory) at init() time. Optional fmt.Stringer for display name. cip.LookupType(code) returns codec; DataType.String() resolves registered names.
- Vendor packages: vendor/rockwell provides TypeCodec wrappers for Timer, Counter, PID, Control. Call rockwell.RegisterTimer(dt), etc. with controller-specific type codes from ListTags.
- Error classification: Modbus protocol errors (IsModbusError) are not retried; transport errors trigger Reset + retry. Same pattern for EIP: cipError not retried, transport errors retried.
- Servers: modbus.NewServer(addr, opts...) and ethernetip.NewServer(router, opts...). Both support net.Pipe injection for testing.
- EIP server has session management (unique handles, validation), ListIdentity/ListServices, client tracking (ConnectedClients()), connect/disconnect callbacks.
- Implicit I/O (cyclic messaging): IOScanner (client) sends Forward_Open, exchanges assembly data over UDP at RPI. Adapter (server) wires ConnMgr callbacks to Runtime for automatic I/O connection setup.
- IOScanner: NewIOScanner(session, udpAddr) -> scanner.Open(ctx, cfg) -> conn.SetOutput(data) / conn.Input() / conn.IsTimedOut() -> scanner.Close(ctx, conn).
- Adapter pattern: AssemblyObject + Runtime (UDP) + Scheduler (5ms tick) + ConnectionManager with WithOnOpen/WithOnClose callbacks wired to Runtime.AddConnection/RemoveConnection.
- Monitor subscribers: mon.NewSubscriber(bufferSize) returns Subscriber with buffered channel. sub.All() is iter.Seq[Event] for for-range. sub.Done() to unregister. Never blocks the monitor.
- plc.Value carries Type (DataType enum) and ByteOrder. Helper methods: val.Bool(), val.Int(), val.Uint(), val.Float32(), val.Float64() decode Raw using the value's byte order.
- Device probe: examples/ethernetip/probe scans identity, TCP/IP interface, Ethernet link, assembly instances, CIP object classes, connection manager stats, and optionally Logix tags.
- Optional Lua bindings (lua/ package, requires github.com/iceisfun/golua/v2 — Lua 5.5.0): industrialLua.Open(v) registers "modbus" and "eip" Lua globals.
- Lua modbus API: modbus.connect(addr, opts) -> client, client:read_holding_registers(addr, qty), client:write_register(addr, val), etc.
- Lua eip API: eip.connect(addr, opts) -> client, client:read_tag(name) -> auto-typed value, client:write_tag(name, val), client:list_tags(), etc.
- Lua methods use colon syntax (client:method(args)); the self parameter is handled automatically.
- PCCC for SLC 500 / MicroLogix (pccc/ sub-package): legacy AB controllers expose no named tags. Address data-table files (N7:0, B3:0/2, F8:5, T4:0.ACC, S:1) and tunnel PCCC inside CIP Execute_PCCC (class 0x67, service 0x4B). PLC-5 word-range commands are out of scope.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iceisfun/goindustrial](https://github.com/iceisfun/goindustrial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
