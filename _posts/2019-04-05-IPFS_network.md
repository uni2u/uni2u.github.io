---
layout: post
title: "IPFS - Network"
categories:
  - IPFS
tags:
  - IPFS
  - identity
lang: ko
author: "uni2u"
meta: "Springfield"
---

# IPFS 디자인 - Network

IPFS는 다양한 기능을 담당하는 하위 프로토콜을 조합하여 구성된다.

- Identities
- Network
- Routing
- Exchange
- Discovery
- Objects
- Files
- Naming

IPFS 를 구성하는 많은 모듈들은 별도의 프로젝트로 진행되고 있다.

-   [IPFS - Content Addressed, Versioned, P2P File System (DRAFT 3)](https://github.com/ipfs/papers/raw/master/ipfs-cap2pfs/ipfs-p2p-file-system.pdf)
-   [github.com/ipfs/specs](https://github.com/ipfs/specs)
-   [github.com/libp2p/specs](https://github.com/libp2p/specs)
-   [github.com/ipld/specs](https://github.com/ipld/specs)

차분히 하나씩 알아보는 시간을 가지고자 한다.

# 1. Network

IPFS의 Network 기능은 독립 프로젝트인 `libp2p` 를 사용하고 있다.

## 1.1 libp2p

P2P Network 를 구축하는데 발생하는 다양한 문제를 해결하는 것을 목적으로 한 라이브러리 모음이다.
[_github.com/libp2p/specs_](https://github.com/libp2p/specs)

IPFS 는 장착된 모듈을 활용하여 내부에서 이용하는 P2P Network 를 구축한다.

공식 구현은 다음과 같다.

- [go-libp2p](https://github.com/libp2p/go-libp2p)
- [js-libp2p](https://github.com/libp2p/js-libp2p)
- [rust-libp2p](https://github.com/libp2p/rust-libp2p)

## 1.2 Transport agnostic

[_3.1 Transport agnostic - github.com/libp2p/specs_](https://github.com/libp2p/specs/blob/master/3-requirements.md#31-transport-agnostic)

P2P 시스템은 Internet 위에 Overlay Network 를 구축한다.  
  
libp2p 는 TCP(UDP)/IP에는 의존하지 않고 Ethernet, Bluetooth 등 다양한 Transport Protocol 에도 마찬가지로 P2P 시스템을 구축하는 것을 목표로 하고있다.

### 1.2.1 multiaddr (self-describing addressing)

자기 기술형 애드레싱 (Adressing)  이라고 할 수 있다.
Address 를 표현하는 문자열에 자신이 이용하고 있는 Protocol 을 포함하는 형식으로 multihash 와 같은 방식을 사용한다.  
  
libp2p 에서는 **multiaddr** 방식이라고 설명하고 있다.

[_multiformats/multiaddr_](https://github.com/multiformats/multiaddr)

```
/<protoName string>/<value string> + <protoName string>/<value string> + ...
```

IPFS 의 Node Address 의 경우 다음과 같이 `<multiaddr>+/ipfs/<Node ID>` 포맷으로 나타낸다.

```
/ip4/127.0.0.1/tcp/9000/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu
# Address :
#   IP          : 127.0.0.1
#   tcp         : 9000
#   IPFS NodeID : QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu
```

multihash 의 특징으로서 ~ over ~ 를 다층적으로 표현할 수 있다.

```
# IPFS over TCP over IPv6 (typical TCP)
/ip6/fe80::8823:6dff:fee7:f172/tcp/4001/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu

# IPFS over uTP over UDP over IPv4 (UDP-shimmed transport)
/ip4/162.246.145.218/udp/4001/utp/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu

# IPFS over IPv6 (unreliable)
/ip6/fe80::8823:6dff:fee7:f172/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu

# IPFS over TCP over DNS4
/dns4/example.com/tcp/4001/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu

# IPFS over TCP over IPv4 over TCP over IPv4 (proxy)
/ip4/162.246.145.218/tcp/7650/ip4/192.168.0.1/tcp/4001/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu

# IPFS over Ethernet (no IP)
/ether/ac:fd:ec:0b:7c:fe/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu
```

![multiaddr 을 활용한 Overlay Networking](/images/ipfs_id04.png)

Layer 마다 Protocol 또는 프록시 구성이 명시적으로 표현되어 있는 것을 알 수 있다.
  
모든 네트워킹 프로토콜이 libp2p 에 의해 구현된 것은 아니다.

multiaddr 자체는 문자열이지만 사용하기 쉽게 하기 위해서는 Parser/Validator 가 필요하다.
  
Node.js 의 경우 [_js-multiaddr_](https://github.com/multiformats/js-multiaddr) 를 이용해 Parse 한다. Validation는 [_js-mafmt_](https://github.com/multiformats/js-mafmt) 가 담당한다.
  
Go의 경우 [_go-multiaddr_](https://github.com/multiformats/go-multiaddr) 를 이용해 Parse 한다.
그 외 multiaddr Instance 를 조작하여 캡슐화/해제 및 터널링을 표현하는 새로운 Instance 를 생성하는 등 편리기능도 가진다.

## 1.3 Multi-multiplexing

[_3.2 Multi-multiplexing - github.com/libp2p/specs_](https://github.com/libp2p/specs/blob/master/3-requirements.md#32-multi-multiplexing)

libp2p 는 Multi-multiplexing (다중-다중화) 개념이 있다.
Multi-multiplexing 관련 설명 부분이 상기 링크뿐인 관계로 명확하게 이해가 되지 않는다.

### 1.3.1 Transports

[![](https://camo.qiitausercontent.com/96df73294a114f7fe50f247cc0b8f27141e5ee5a/68747470733a2f2f7261772e67697468756275736572636f6e74656e742e636f6d2f6c69627032702f696e746572666163652d7472616e73706f72742f6d61737465722f696d672f62616467652e706e67)](https://github.com/libp2p/interface-transport)

Peer 간 접속을 확립할 때 Transports 라는 인터페이스 사양으로 정의되어 있다.  
TCP, UDP, WebRTC, WebSocket 등의 다양한 Transport Protocol 을 사용 가능하게 하는 것을 목적으로 하고 있다.

[_libp2p/interface-transport_](https://github.com/libp2p/interface-transport)

![libp2p Transports](/images/ipfs_id06.png)

- Client
  - Dial (Client ) 기능
  - `multiaddr` 에 의한 접속처 Peer 지정이 가능
  - 돌려받는 Connection 은 interface-connection 을 제공하여야 함
- Server
  - Listen (Server) 기능
  - Handler 에 Connection 은, interface-connection 을 제공하여야 함
- 인스턴스화
  - 예를 들어 WebRTC 의 Signaling 정보는 Dial/Listen 에서 공통적으로 이용하므로 인스턴스화하여 상태를 보관

### 1.3.2 Connection

[![](https://camo.qiitausercontent.com/dc9bc56a8c4f7e7fed991cc41a9982340f618b2e/68747470733a2f2f7261772e67697468756275736572636f6e74656e742e636f6d2f6c69627032702f696e746572666163652d636f6e6e656374696f6e2f6d61737465722f696d672f62616467652e706e67)](https://github.com/libp2p/interface-connection)

Transporter 에 의해 작성된 Connection 의 action 은 Connection 인터페이스 사양으로 정의되어 있다.

[_libp2p/interface-connection_](https://github.com/libp2p/interface-connection)

- Read/Write
- Full/Half Duplex 서포트
- 플로우 제어
  - Back-Pressure
- 절단 처리
  - Half-Close

Transports 와 Connection 을 구성한 라이브러리는 아래에 정리되어 있다.

[_Transports - libp2p.io/implementations_](https://libp2p.io/implementations/)

- libp2p-tcp
- libp2p-quic ( Stream Muxer 含む)
- libp2p-websockets
- libp2p-webrtc-star
- libp2p-webrtc-direct
- libp2p-udp
- libp2p-utp

**Go**

Go 의 libp2p Interface 는 [go-libp2p-net](https://github.com/libp2p/go-libp2p-net/blob/master/interface.go) 프로젝트에서 정의하고 있다.

Transport 에 관한 interface 는 [go-libp2p-transport](https://github.com/libp2p/go-libp2p-transport) 모듈에 정의하고 있다. (예를 들어 접속을 나타내는 `Conn` 은 `libp2p-net/Conn` 을 확장)  
즉, 각 Transport 구현은 `go-libp2p-transport` 의 Interface 를 구현하는 형태가 된다.

**다른 Node 에 접속요구** 또는 **다른 Node 로부터 접속 요구를 수신**해 확립된 접속 (ex. `net.Conn`, `websocket.Conn`) 은 ① [go-multiaddr-net](https://github.com/multiformats/go-multiaddr-net) 의해 multiaddr friendly 한 API로 랩핑 된 후, ② [go-libp2p-transport-upgrader](https://github.com/libp2p/go-libp2p-transport-upgrader) 의해 **Stream 다중화 가능 통신의 암호화** 된 Connection으로 Upgrade (후술)된다.

![libp2p Transports Interfaces](/images/ipfs_id07.png)

#### 1.3.2.1 Connection Manager

네트워크 자원을 관리하기 위해서는 전체 Connection 을 종합적으로 감시하는 역할이 필요하다.  
이를 위한 역할이 Connection Manager 이다.

![Connection Manager](/images/ipfs_id08.png)

- 접속 Node 수 제한  
- 대역 제한
  - 수신/송신별 제한
- Latency 감시

**구현**
- Node
  - [_js-libp2p-connection-manager_](https://github.com/libp2p/js-libp2p-connection-manager)
- Go
  - [_go-libp2p-interface-connmgr_](https://github.com/libp2p/go-libp2p-interface-connmgr)  (Interface only)

#### 1.3.2.2 Filter

접속하고 싶지 않은 Node 가 있는 경우 Filter 를 설정할 수 있다.  
Filter 대상은 multiaddr 에 의해 지정된다.

```
// make a new filterset
f := NewFilters()

// filter out addresses on the 192.168 subnet
_, ipnet, _ := net.ParseCIDR("192.168.0.0/16")
f.AddDialFilter(ipnet)

// check if an address is blocked
lanaddr, _ := ma.NewMultiaddr("/ip4/192.168.0.17/tcp/4050")
f.AddrBlocked(lanaddr) // = false  
```

**구성**
- Node
  - Transport Class 는 공통으로 `filter` 메소드를 제공
- Go
  - [_go-maddr-filter_](https://github.com/libp2p/go-maddr-filter)
  - [_multiaddr-filter_](https://github.com/whyrusleeping/multiaddr-filter): CIDR 형식의 넷 마스크를 multiaddr 로 작성
    - `ipnet, _ := multiaddrFilter.NewMask("/ip4/192.168.0.0/ipcidr/16")`

### 1.3.3 Network Layer

libp2p 에서 Network Protocol 의 구성은 역할별로 몇가지 Layer 로 나뉘어 있다.

TCP/IP 계층 모델의 경우 (OSI 모델에 대략 대응하는 것),

- Link (물리층, 데이터링크)
- Internet (네트워크)
- Transport
- Application (세션, 프레젠테이션, 어플리케이션)

라고 하는 Layer 를 형성하고 있다.
송신 데이터그램은 Upper 에서 시작되어 Lower 를 향해서 Encapsulation 되며 수신은 Lower 에서 시작되어 Upper 를 향해서 Decapsulation 가 된다.

![Encapsulation 과 Decapsulation](/images/ipfs_id09.png)

libp2p 에서는 대략 다음과 같은 Layer 가 형성되고 있다.

- Transport Layer
- Encryption Layer
- Stream Multiplex Layer
- Application Layer

![Transport/Encryption/Stream Multiplex/Appliation Layer 동작](/images/ipfs_id10.png)

- Transport Layer
  - 다른 Peer 와 Connection 구성
  - Protocols
    - TCP
    - UDP, uTP
    - WebSocket
    - WebRTC
- Encryption Layer
  - Connection 상의 데이터그램을 암호화 및 서명 추가
  - Protocols
    - secio
- Stream Multiplex Layer
  - Connection 상에 다수의 Stream 을 가상으로 확립
  - Protocols
    - SPDY, HTTP/2
    - yamux
    - mplex
- Application Layer
  - Application이 이용
  - Protocols
    - DHT
    - BitSwap

libp2p 에서는 Encapsulation 하여 Layer 를 올리는 것을 Upgrade 라고 한다.

**Go 구성**

Go의 경우 쌍방향 통신을 실시하는 경로 (Connection, Stream) 는 모두 `ReadWrite Closer` 에 추상되어 각 Layer 의 랩핑으로 실현한다.

![libp2p Layer 구성](/images/ipfs_id11.png)

![libp2p Layer 세부 동작](/images/ipfs_id12.png)

```
// transport connection
type Conn io.ReadWriteCloser
func (c *Conn) Write(data []byte) error {
  sendMesage(data)
}

// encrypted encapsulation
type encryptedConn {
  conn Conn
}
func (e *encryptedConn) Write(data []byte) error {
  header := createheader(data)
  encrypted := encrypt(data)
  buf := concat(header, encrypted)
  e.conn.Write(buf)
}
func wrapEncryptConn(insecure io.ReadWriteCloser) (*encryptedConn, error) {
  return &encryptedConn{conn: insecure}
}

// stream multiplex encapsulation
type muxedConn {
  conn encryptedConn
}
func (m *muxedConn) Write(data []byte) error {
  header := createheader(data)
  buf := concat(header, data)
  m.conn.Write(buf)
}
func wrapMuxedConn(simple io.ReadWriteCloser) (*muxedConn, error) {
  return &muxedConn{conn: simple}
}

conn, err := transportConn(multiaddr)
econn, err := wrapEncryptConn(conn)
mconn, err := wrapMuxedConn(econn)

mcon.Write(bufferarray)
```

### 2.3.4 Encryption

통신의 기밀성, 무결성을 담보하기 위해서는 통신 내용의 암호화 및 변조 감지가 필요하다.

Transport Protocol 에 의해 암호화되어있는 것도 있지만 Transport Protocol 에 의존하지 않고 암호화 및 변조 검출을 위해 이 레이어에서 다시 암호화 및 변조 감지 제공을 한다.

![Encryption Layer](/images/ipfs_id13.png)

PeerID 생성시에 공개 키를 생성하고 있기 때문에 TLS Like 암호화를 하는 것은 어렵지 않다.

![TLS 암호화](/images/ipfs_id14.png)

**구성**

Crypto channels 라이브러리는 아래에 정리되어 있다.

[_Crypto channels - libp2p.io/implementations_](https://libp2p.io/implementations/)

- libp2p-secio

Go 의 경우 관련 interface가 [go-conn-security](https://github.com/libp2p/go-conn-security) 라는 모듈에 정의되어 있다.  
현재 secio 만 구현되어 있어 libp2p 의 디폴트라 할 수 있다.

Go 의 경우 Upgrader 에 의해 Transport 의 Connection 을 Upgrade 하는 형태로 적용된다.  
Upgrade 된 Connection 을 사용하여 데이터 송신/수신하는 경우 모든 데이터 그램이 이 Layer 의 Encapsulation/Decapsulation 에 의해 수송된다.

![Encryption](/images/ipfs_id15.png)

Todo
