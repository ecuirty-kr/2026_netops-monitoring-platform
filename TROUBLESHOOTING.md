##  트러블슈팅 로그 (진행하며 계속 업데이트)

프로젝트 진행 중 겪은 issue와 해결 과정 기록

```
## [26-07-13] GNS3 Setup Wizard에서 "Could not find a VM named 'GNS3 VM'" 에러

**증상**: GNS3 Setup Wizard 진행 중 VirtualBox/VMware에서 GNS3 VM은 찾지 못함

**원인**: GNS3 VM.ova 파일을 VirtualBox에 아직 임포트하지 않은 상태에서
          Setup Wizard를 실행함.
          Virutalization software 선택이 VMware로 되어 있어 환경 불일치 확인함

**해결**: VirtualBox에서 File > Import Appliance로 GNS3 VM.ova 수동 임포트 후,
          Setup Wizard에서 Virtualization software를 VirtualBox로 명시적 선택
          위 설정 후 재시도

---

## [26-07-15] GNS3 GUI에서 "KVM accleration cannot be used (/dev/kvm doesn't exist)." 에러

**증상**: GNS3 캔버스에서 VyOS 노드 Start 진행 불가

**원인**: VirtualBox에서 VyOS 실행하기 위한 KVM(리눅스 커널 자체 가상화 가속 기술)이 비활성화
          되어 있음.
          GNS3 VM이 VirtualBox 위에서 가동되는 가상 머신이라 중첩 가상화가 필요하기 때문.

**해결**: VirtualBox의 GNS3 VM 콘솔 창을 통해 gns3_server.conf 파일 내용 수정, [Qemu]
          enable_kvm = false 내용 추가 및 저장 후 GNS3 VM 재부팅

---

## [26-07-15] GNS3 GUI에서 "POST /projects/d19aecf3- .../qemu/nodes/.../start not found" 에러

**증상**: GNS3 캔버스에서 VyOS 노드 Start 진행 불가. vyOS 삭제 시 DELETE not found 에러 발생

**원인**: GNS3 VM 재부팅하면서 서버측 프로젝트 및 노드 상태가 초기화 되었으나 GNS3 로컬 프로그램은
          이전 프로젝트 및 노드 ID를 사용하고 있어 불일치 발생. (404 Not Found 와 유사함)
          서버가 재시작되면서 GUI와 설정 동기화가 꼬임.

**해결**: GNS3 프로그램을 완전히 종료한 뒤 다시 실행 후 vyOS 노드 Start 재시도


---

## [26-07-16] KVM 비활성화로 인한 VyOS 부팅 속도 저하 [개선 예정]

**증상**: enable_kvm = false 설정 후 VyOS 노드 부팅에 15분 이상 소요, 
          GNS3 VM CPU 사용률 100% 지속 확인

**원인**: KVM 없이 소프트웨어 에뮬레이션만으로 QEMU를 돌리고 있어 CPU 부담이 큼

**개선 계획**: VirtualBox의 Nested VT-x/AMD-V 옵션 활성화 후 KVM 재활성화하여 성능 비교 예정


---

## [26-07-16] VyOS 부팅 중 커널 패닉 발생 "Kernel panic - IO-APIC + timer"

**증상**: enable_kvm = false 적용 후 VyOS 노드 부팅 시, Console 창에서
          "Kernel panic - not syncing: IO-APIC + timer doesn't work!" 메시지 출력 및 부팅 실패

**원인**: KVM 가속 없이 순수 소프트웨어 에뮬레이션(QEMU)으로 인터럽트 컨트롤러(APIC)를
          처리하면서 타이머 동기화 실패

**해결**: VirtualBox의 GNS3 VM 설정에서 시스템 - 프로세스 - Nested VT-x/AMD-V 가상화 활성화,
          enable_kvm = true로 변경하여 하드웨어 가속 기반으로 전환

---

## [26-07-16] Hyper-V(WSL2)와 VirtualBox 중첩 가상화 충돌

**증상**: VirtualBox의 Nested VT-x/AMD-V를 활성화 했으나 GNS3 VM에서
          "KVM is not supported! ... another virtualization solution is already running"
          에러 메시지 발생

**원인**: PowerShell bcdedit 확인 결과 hypervisorlaunchtype이 Auto로 설정되어 있음.
          이전 CKA 실습 중 설치한 WSL2의 Hyper-V 기반으로 현재 활성화되어 있어
          VirtualBox의 중첩 가상화와 충돌함

**해결**: WSL2를 유지, Hyper-V는 유지하되 GNS3 실행 방식 변경
          (VirtualBox 기반 GNS3 VM 로 계획 수정 검토)

---

## [26-07-16] GNS3 GUI 에서 WSL2 서버(localhost:3080) 연결 실패

**증상**: WSL2에 GNS3 서버 정상 실행 확인, Windows에 GNS3 GUI에서 localhost:3080 접속 시도 시
          "Cannot connect to the GNS3 server" 에러 발생.
          PowerShell에서 curl http://localhost:3080/v2/version 연결 실패.

**원인**: WSL2의 locaohost 포트 포워딩이 정상 동작하지 않음.
          WSL2 터미널에서 ip addr show eth0로 WSL2 실제 IP(172.18.137.64) 확인 후,
          IP 주소로 curl 시도 시 정상 응답(200 OK) 확인

**해결**: GNS3 GUI의 Main server 설정에서 Host를 localhost 대신 WSL2의 실제 IP 주소로 변경.
          Windows 재부팅 시 IP 변경될 수 있으므로 확인 필요.


---

## [26-07-17] VyOS 노드 KVM 권한 에러 (Permission Denied)

**증상**: WSL2 환경에서 VyOS 노드 Start 시
          "Could not access KVM kernel module: Permission denied", "uBrindge is not available"
          에러 발생

**원인**: WSL2 사용자 계정이 kvm 그룹에 속해있지 않음.
          groups 명령어로 확인 시 kvm 그룹 자체는 시스템에 존재했으나 사용자 계정이 포함되지 않음


**해결**: sudo usermod -a -G kvm $USER 로 그룹 추가 후,
          PowerShell에서 wsl --shutdown으로 WSL2 세션 재시작하여 그룹 권한 반영.
          이후 groups 명령어로 kvm 계정 포함된 것 확인, gns3server 재실행하여
          VyOS 노드 정상 부팅 확인 (Stop 후 Start, 부팅 시간 약 5초 후 KVM 가속 정상 작동 확인)


---

## [26-07-17] GNS3 노드들이 링크 연결된 상태에서 QEMU Network 어댑터 개수 변경 시도 시 에러

**증상**: VyOS-1 노드의 Adapters 값을 1에서 4로 변경할 때,
          "Changing the number of adapters while links are connected isn't supported yet!
          Please delete all the links first." 에러 발생

**원인**: GNS3 노드에 링크(케이블)가 연결된 상태에서는 어댑터 개수 변경을 지원하지 않음.
          노드 실행 상태(Stop 여부)와 무관하게 연결된 링크 자체를 먼저 삭제 후 진행해야 함.

**해결**: 연결된 링크를 모두 삭제한 뒤 Network Adapters 값을 4로 변경 후 다시 링크 연결.
          또한 템플릿의 기본 Adapters 값 변경 시 캔버스에 이미 배치된 기존 노드에는
          적용되지 않아, 노드별로 개별 Configure에서 수동으로 값 변경이 필요함

---

## [26-07-18] GNS3 Cloud 노드로 WSL2 호스트 네트워크 연결 실패

**증상**: GNS3 Cloud 노드를 통해 VyOS eth2와 WSL2를 같은 대역 IP(172.18.137.100/20)로
          수동 설정했으나, WSL2에서 ping 시도 시 "Destination Host Unreachable" 에러 발생.
          VyOS 라우팅 테이블에는 정상적으로 directly connected 경로 잡혀있는 상태 확인함
 
**원인**: WSL2 자체가 Hyper-V 기반 가상 환경이라 eth0 연결이 이미 가상 인터페이스임.
          GNS3 Cloud 노드는 물리 인터페이스 브릿지 방식에 가깝기 때문에
          이중 가상화 계층에서 정상적으로 브릿지가 이루어지지 않음

**해결**: Cloud 노드 대신 GNS3의 NAT 노드로 변경하여 진행함.
          NAT 노드는 GNS3 서버가 자체 관리하는 독립적인 가상 네트워크(192.168.122.0/24)를
          제공하여, VyOS에서 DHCP로 IP를 할당받아 WSL2와 정상 통신 확인 (ping 성공)

---

## [26-07-18]  VyOS-2, VyOS-3 SNMP 등록 전 멀티홉 구간 통신 불가 (라우팅/방화벽/ARP 복합)

**증상**: VyOS-1은 NAT 노드 경유로 Zabbix 서버(WSL2)와 SNMP 통신이 정상적으로 이루어졌으나,
          VyOS-2(10.0.0.2), VyOS-3(10.0.1.3)는 WSL2 터미널에서 ping/snmpwalk 시도 시
          100% timeout 발생

**원인**: 아래 3가지 요인이 순차적으로 얽혀있었음.
          1. WSL2(192.168.122.0/24)에 VyOS-2/3 대역(10.0.0.0/24, 10.0.1.0/24)으로 가는
             리턴 라우트가 없어 요청 자체가 목적지에 도달하지 못함
          2. libvirt가 virbr0(NAT 네트워크) 생성 시 자동 삽입되는 LIBVIRT_FWI/FWO
             iptables 규칙이 192.168.122.0/24 외 트래픽을 REJECT 하고 있음
             (확인 결과, 로컬에서 보낸 ping은 FORWARD 체인을 타지 않아 실제 원인이 아니었음)
          3. VyOS-1의 ARP 캐시에 VyOS-2/3의 MAC 정보가 등록되지 않아
             최초 요청 시 ARP 해석 과정에서 지연이 발생하여 패킷 유실 발생

**해결**: 1. WSL2 에 정적 라우트 추가
          - sudo ip route add 10.0.0.0/24 via 192.168.122.186
          - sudo ip route add 10.0.1.0/24 via 192.168.122.186
          2. ARP는 트래픽 발생 시 자동으로 해석되어 특별한 조치 없이 자연 해결됨.
          3. tcpdump로 eth2/eth0 양쪽 인터페이스를 GNS3 콘솔(다중 탭 미지원) 및 SSH로
             별도 세션을 확보하여 동시 모니터링하여 구간별 원인을 파악하며 좁혀나감.
          (※ WSL2 재부팅 시 라우트 유실됨. 영구 적용은 추후 검토)

---

## [26-07-18] VyOS static route 설정이 FRR(FIB)에 미반영

**증상**: `set protocols static route 0.0.0.0/0 next-hop 10.0.0.1(10.0.1.1)` commit 후 save 완료.
          `show configuration commands | match static` 명령으로 설정을 확인했으나,
          `run show ip route`, `vtysh` 접속 후 `show ip route`에서 해당 설정이 보이지 않음.
          `restart frr`로 FRR 데몬 재시작 이후에도 동일한 현상 지속됨

**원인**: `run show log tail 100 | match static`으로 로그 확인 결과,
          VyOS 최초 부팅 시점에 staticd가 초기 연결에 실패한 이력 확인
          (watchfrr: staticd state -> down : initial connection attempt failed)
          Live-ISO 환경 특성상 설정 반영 과정에서 발생한 오버레이 파일시스템 이슈로 추정됨.
          (vtysh write 시 "Error renaming /etc/frr/frr.conf ... Device or resource busy" 에러)
          VyOS CLI -> FRR로의 config 동기화가 에러 발생 없이 실패하는 것으로 예상.

**해결**: vtysh로 FRR에 라우트를 직접 적용하여 VyOS 설정 동기화 계층을 우회함.
          단, VyOS config tree에는 반영되지 않아 `show configuration` 출력으로 확인되지 않음.
          Live-ISO 특성상 재부팅 시 유지되지 않으므로 재설정이 필요할 수 있음

---

## [YY-MM-DD]

**증상**: ...
**원인**: ...
**해결**: ...
