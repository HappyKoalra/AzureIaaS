### Azure에서 제공하는 [가상 네트워크 게이트웨이의 기본 P2S(Point to Site) VPN 구성](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/point-to-site-about)를 통해 가상 네트워크에 개발자/엔지니어가 안전하게 네트워크 연결을 이용할 수 있습니다.

Azure의 기본 P2S VPN 구성이 아닌 별도의 가상 머신을 생성하여, 오픈 소스 기반의 OpenVPN을 구성할 수도 있습니다. 
OpenVPN을 가상 머신에 구성하려면 아래의 단계가 필요합니다.

이 아티클에서 사용한 가상 네트워크 및 OpenVPN 네트워크 구성은 아래와 같습니다.
>* Azure 가상 네트워크 IP 주소 대역 192.167.0.0/24
>* OpenVPN Server IP 주소 192.167.0.4
>* 라우팅이 가능한지 확인하기 위한 Windows Server IP 주소 192.167.0.5

1. **Linux 기반**의 가상 머신을 생성

2. 별도의 추가 설정이 필요한 부분은 해당 Linux VM의 NIC에 대해 IP Forward 구성이 필요합니다. Guest OS 단이 아닌 Azure Hyper-V 호스트의 VM 구성에서 NIC을 P-Mode로 변경, 해당 VM IP 주소로 향하는 패킷과 더불어, 다른 주소 패킷도 수신을 가능하게 합니다. VPN 구성뿐만 아니라, Azure내에서 Router나 보안 어플라이언스 구성시에도 필요한 옵션입니다.

![IP Forwarding Enable](/Network/Images/OpenVPN-01.png "NIC IP 전달 사용")

3. 해당 Linux에 [OpenVPN을 설치](http://www.startupcto.com/server-tech/centos/setting-up-openvpn-server-on-centos)합니다. 기본적으로 EasyRSA를 사용하여, Key를 생성합니다. 더불어 server.conf를 구성, OpenVPN 클라이언트에서 키와 ovpn의 설정은 기존 OpenVPN 구성과 동일합니다.

4. OpenVPN의 server.conf의 구성 중 확인해야 할 사항은 아래와 같습니다.
* VPN 연결용 네트워크 - 10.6.0.0/24

![server.conf-01](/Network/Images/OpenVPN-02.png)
* 192.167.0.0/24를 VPN 클라이언트의 라우팅 경로로 전달

![server.conf-02](/Network/Images/OpenVPN-03.png "server.conf-02")
* /etc/sysctl.conf내 ip_forward 구성

![server.conf-03](/Network/Images/OpenVPN-04.png "server.conf-03")

5. OpenVPN 서버에 구성된 클라이언트 Key와 ovpn 설정을 클라이언트에 전달하여, OpenVPN 서버로 접속되는지 확인합니다. 이 단계까지는 Azure에서 별도의 라우팅 테이블 수정없이도, 확인이 가능합니다.

![OpenVPN 클라이언트](/Network/Images/OpenVPN-05.png "OpenVPN 클라이언트")

![클라이언트의 Route Print 명령어](/Network/Images/OpenVPN-06.png "Route Print")

![ICMP, SSH 확인](/Network/Images/OpenVPN-07.png "ICMP, SSH")

6. 이후 Azure 가상 네트워크 내 다른 서버로 연결하려면, Azure 가상 네트워크의 라우팅 테이블 수정이 필요합니다. Azure의 경우에는 개별 라우팅은 별도 지정을 하지 않으면, 가상 네트워크 서브넷 이외의 네트워크 경로에 대해서 Azure Network에서 결정합니다. 대표적으로 인터넷 연결의 경우 개별 VM에서 Azure SDN Gateway를 통해 연결합니다. 이에 대한 라우팅 테이블 반영이 필요합니다. Azure 포탈에서 경로 테이블을 이용합니다. 빈 테이블을 생성하고, 하나씩 채워넣게 되어 있습니다.

![경로 테이블](/Network/Images/OpenVPN-08.png "경로 테이블")

7. 추가를 클릭하면, 이름 및 경로, Hop을 구성할 수 있습니다. 이 때, 10.6.0.0/24에 대해서는 Azure SDN이 네트워크 라우팅을 처리하지 않고, 별도의 장치에서 처리한다고 선언해야 합니다. 해당 아티클내 테스트의 경우엔 10.6.0.0/24 경로에 대해서, 192.167.0.4로 전달합니다.

![경로 편집](/Network/Images/OpenVPN-09.png "경로 편집")

8. 이를 OpenVPN 클라이언트들이 라우팅하여 접속할 가상 네트워크내 서브넷에 연결합니다.

![경로 연결](/Network/Images/OpenVPN-10.png "경로 연결")

9. 반영에 대략 30초정도가 필요합니다. 이후, 가상 네트워크 내 다른 서버에 접속을 시도하면, 연결이 됩니다. (192.167.0.5의 Windows Server. Windows 방화벽은 Off해놓았습니다.)

![연결 확인](/Network/Images/OpenVPN-11.png "연결 확인")