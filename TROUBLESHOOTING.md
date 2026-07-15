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

## [YY-MM-DD] 제목

**증상**: ...
**원인**: ...
**해결**: ...
