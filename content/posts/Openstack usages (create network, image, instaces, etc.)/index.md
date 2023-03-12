---
title: '7. Openstack usages (create network, image, instaces, etc.)'
date: 2021-01-02
weight: 7
layout: 'list'
---
---
> Specification : openstack, internal & external networks, flavor, images, instances, key, security groups

### **Summary step** 

 1. Create external network
 2. Create router
 3. Create internal network
 4. Add internal network to router
 5. Create images
 6. Create flavor 
 7. Create Instances & key
 8. Add floating IP
 9. Create security groups
 10. Add SG to instances
 11. Verify VM connectivity

&nbsp;

---

&nbsp;
&nbsp;

### **Step 1 by 1** 

#### 1. Create external network 
Goto admin pages, and please follow guide correctly 

![aio-yoga](./images/1.png)

Make **Physical Network** use **physnet1**

![aio-yoga](./images/2.png)

Add **Network address** by using **ens4** network address, because for external & floating IP, make sure this network can connect to internet.

![aio-yoga](./images/3.png)

#### 2. Create router

Add router 

![aio-yoga](./images/4.png)

Make sure **external network** using previously created

![aio-yoga](./images/5.png)

#### 3. Create Internal network
This network for instances private IP, you can use custom address


![aio-yoga](./images/6.png)
![aio-yoga](./images/7.png)

You can choose anything private address :

![aio-yoga](./images/8.png)
![aio-yoga](./images/9.png)

#### 4. Add internal network to router
Back again to router menu, and add internal network on router like bellow :
![aio-yoga](./images/10.png)
![aio-yoga](./images/11.png)
Make sure network correct :

![aio-yoga](./images/12.png)

#### 5 & 6, Create images & Flavor

We use lightweight image with cirros 

```
wget http://download.cirros-cloud.net/0.5.2/cirros-0.5.2-x86_64-disk.img
```

![aio-yoga](./images/13.png)


**Create image**
```
openstack image create \
    --container-format bare \
    --disk-format qcow2 \
    --file cirros-0.5.2-x86_64-disk.img \
    cirros-img
```

**Create flavor**

```
openstack flavor create --ram 512  --vcpus 1 --disk 10 m1.tiny
```


![aio-yoga](./images/14.png)

#### 7. Create instances

![aio-yoga](./images/15.png)

add VM name
![aio-yoga](./images/16.png)

Chosse cirros image
![aio-yoga](./images/17.png)

Choose flavor
![aio-yoga](./images/18.png)

Choose internal network 
![aio-yoga](./images/19.png)

Create new keypair or you can paste pubkey
![aio-yoga](./images/20.png)


#### 8. Add floating IP

Click on instances, choose floating ip address

![aio-yoga](./images/21.png)
![aio-yoga](./images/22.png)

For pool, you need use external network 
![aio-yoga](./images/23.png)

Yeay, you already have floating IP
![aio-yoga](./images/24.png)


#### 9. Create security group
security group like firewall, you can open or closed port, by default deny all, we need create first to open

Go to security groups menu

![aio-yoga](./images/25.png)

Add rule for allow, bellow example allow all ICMP
![aio-yoga](./images/26.png)

allow all TCP (Only for test, better open by need)
![aio-yoga](./images/27.png)

#### 10. Add SG to instances

![aio-yoga](./images/28.png)
![aio-yoga](./images/29.png)

#### 11. Verify VM connectivity
Bellow you can see, we can access VM using Floating IP & VM can ping to internet :)


![aio-yoga](./images/30.png)


## Thankyou