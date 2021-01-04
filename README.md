# Nmap Essentials

## What Is Nmap

**Nmap**, short for Network Mapper, is a free, open-source tool for vulnerability scanning and network discovery. Network administrators use **Nmap** to identify what devices are running on their systems, discovering hosts that are available and the services they offer, finding open ports and detecting security risks.

## Installing Nmap

{% tabs %}
{% tab title="Linux" %}
#### Using RPM To Install

Many popular Linux distributions \(Redhat, Mandrake, Suse, etc\) use the [RPM](http://www.rpm.org/) package management system for quick and easy binary package installation, now we go to the Terminal and type.

```javascript
rpm -vhU https://nmap.org/dist/nmap-7.91-1.x86_64.rpm
```

#### Using Kali/Parrot Linux

If you want the tools already installed on your system you can down the Security Versions which can be found here.

* [Kali Security](https://www.kali.org/downloads/)  
* [Parrot Security](https://www.parrotsec.org/download/) \(Click Security Tab\)  

  Or use this command to install the Nmap Installer and install it automatically,

```javascript
git clone https://github.com/nmap/nmap.git
```
{% endtab %}

{% tab title="Windows" %}
[https://nmap.org/book/inst-windows.html](https://nmap.org/book/inst-windows.html)
{% endtab %}
{% endtabs %}

## Using Nmap

To start use `nmap` this will say that we are using the Nmap module and then we can start adding options then the IP Address we want to scan, the options can be specifyed using either `-option` or `--option` depending on what one your doing and then the IP Address after the options.

### Example \(Normal Speed\)

```text
nmap 10.10.10.10
```

### Example \(Fast Scan\)

```text
nmap -F 10.10.10.10
```

