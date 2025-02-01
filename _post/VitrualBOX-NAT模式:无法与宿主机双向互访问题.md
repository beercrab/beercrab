---
日期: 2024年12月23日
---

最近在部署一台web服务器，考虑了下使用虚拟机部署，由于长期使用VMware Workstaion(以下简称:VMware)部署虚拟机，这次想换个可替代的虚拟化产品，就选择了VitrualBox(以下简称:VBox)这款被Oracle收购的开源虚拟化产品，使用后发现体验并不理想，先说下本次部署应用的软件及其版本：

- 软件版本:
  宿主机:windows 10
  虚拟机软件:VitrualBox 7.1
  虚拟机客户机:Debian 12

- 网络:
  宿主机:192.168.2.30/24
  虚拟机客户机:203.0.113.129/24
 
通常，VMware的客户机设置完NAT后，宿主机既可以Ping通客户机，也可以访问客户机中任何服务，而VBox的客户机网络模式同样设置NAT的情况下，宿主机无法ping通客户机，且无法访问访问客户机中任何应用服务，但是客户机却能正常访问外部网络和服务

![Image](https://github.com/user-attachments/assets/2392923f-0a17-4df5-8428-38d54e9a9311)

为了解决VBox的NAT这个问题，查阅官方产品使用手册，手册全英文，我结合翻译再加上实际环境遇到的问题，就简单描述下VBox的NAT工作机制。
VBox中的NAT可以认为是台虚拟路由器，是VBox核心网络引擎，在VBox中，这台虚拟路由器的逻辑位置在虚拟机和宿主机之间(具体上图示意图)
VBox的NAT模式有个特点也是缺点，与真实环境中路由器网络非常相似，虚拟客户机对于外部网络是不可见且无法访问，即使宿主机也一样。关于这个问题，官方也提供了解决办法，**使用NAT配置端口转发**，即可实现宿主机访问客户机的网络服务

关于VBox NAT的一些使用上的限制，官方给了说明，需要注意以下几点：

- ICMP协议限制:比如ping、tracertroute这类工具可能会失去作用
- 仅支持特定协议:仅支持TCP/UDP，不支持GRE、VPN、DDNS
- NAT端口转发:不支持公共端口(0-1024)

最后再看下Vmware的NAT工作机制，Vmware虚拟机软件安装到Windows宿主机系统时，系统会生成一个VMnet8的虚拟网络适配器，客户机在NAT模式中访问外部网络时与VBox中的机制差不多，唯一不同的是VMware中宿主机可以直接访问虚拟客户机，且不需要依托端口转发。因为宿主机和虚拟客户机共用VMnet8这个虚拟适配器，所以主机系统可以通过NAT直接与虚拟客户机相互通信。

关于Vmware NAT的一些使用上的限制，官方也给了说明，需要注意以下几点：

- NAT并不完全透明。尽管可以手动配置NAT设备来建立虚拟机与互联网服务器连接，但NAT通常不允许从外部互联网想虚拟机发起连接。在实际环境中，这会导致一部分需要从外部互联网向虚拟机发起连接的TCP和UDP协议无法自动运行或根本不运行。(比如DDNS)
- NAT能提供一些防火墙保护，比如仅允许虚拟机通过NAT访问外部网络，而外部网络无法访问虚拟机
- NAT中虚拟机可以与宿主机双向访问，但是无法与宿主机相同子网的其他外部主机相互访问，除非启用NAT的端口转发

上面两个虚拟化产品中NAT模式的还有个共同缺点：NAT模式会造成性能上的损失，如果宿主机硬件性能充沛，这类损失所造成的影响极为有限。

所以两款虚拟机NAT网络模式限制非常多，并不适用复杂的网络环境，最佳建议：首选虚拟机的桥接网络模式

附:官方资料
- [VirtualBox中的虚拟网络](https://www.virtualbox.org/manual/topics/networkingdetails.html#nat-limitations)
- [VMware Workstation Pro中配置网络地址转换](https://docs.vmware.com/cn/VMware-Workstation-Pro/17/com.vmware.ws.using.doc/GUID-89311E3D-CCA9-4ECC-AF5C-C52BE6A89A95.html)
- [VMware Workstation Pro中NAT配置的功能和限制](https://docs.vmware.com/cn/VMware-Workstation-Pro/17/com.vmware.ws.using.doc/GUID-122B85E2-E60F-4C44-87B2-A3F8DDC88D66.html)
