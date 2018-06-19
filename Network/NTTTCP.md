### Azure에서 제공하는 가상 머신들은 사용하는 크기(SKU)에 따라 사용 가능한 NIC의 개수와 처리량이 다르게 설계되어 있습니다.

가상 머신 크기에 대한 문서는 [여기](https://docs.microsoft.com/ko-kr/azure/virtual-machines/windows/sizes)를 참고합니다. 지역별로 제공하는 가상 머신 크기는 상이하므로, 정확한 크기를 확인하고자 할 경우, 아래 *PowerShell*을 통해 가능합니다.
```azurepowershell
Get-AzureRmLocation # 서비스를 제공하고 있는 지역 확인
Get-AzureRmVMSize -Location '<지역>'
```

![가상 머신 SKU](/Network/Images/NIC_QOS.PNG "가상 머신 SKU")

위의 그림에 따르면, Standard_F2s_v2의 경우에는 최대 2개의 NIC, 그리고 875Mbps(b가 소문자 이기에 bit입니다. Byte로 계산하시려면, 8을 나눠줘야 합니다.)까지 제공합니다. 이 이상을 요청할 경우에는, 서비스 제공 제한에 도달하여, Throttling이 걸리게 됩니다. 이는 개별 NIC의 제공 값이 아니라, 전체 합임에 주의해야 합니다.

그렇다면 정말, 해당 대역폭을 제공하는지에 대한 테스트를 하시고 싶은 경우가 있을 수 있습니다. [NTTTCP](https://gallery.technet.microsoft.com/NTttcp-Version-528-Now-f8b12769)를 사용하여, 테스트가 가능합니다.

테스트와 관련된 가이드는 [대역폭/처리량 테스트(NTTTCP)](https://docs.microsoft.com/ko-kr/azure/virtual-network/virtual-network-bandwidth-testing)를 통해서 살펴보실 수 있습니다.