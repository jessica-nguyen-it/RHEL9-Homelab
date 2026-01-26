# RHEL 9 SysAdmin Homelab 🐧

This homelab is where I test, break, fix, and learn everything I need for the RHCSA. I’m documenting my progress as a multi-part series, with each entry focused on a specific exam domain. I’ll be updating regularly as I work through new topics, break things (on purpose or not), and learn from the process. 

```Now with all that context out of the way, let’s get to what you're actually here for!```

## First: the practice environment(s)

I initially focused my practice on an AWS EC2 instance, but over time I expanded the project to address different needs. I incorporated VMware to cost‑effectively replicate the RHCSA test environment and leveraged my ThinkPad for casual, daily practice. Altogether, this setup really helped me grow more confident working across cloud, virtualized, and local platforms.

```
1. AWS EC2 (Cloud, 1 VM)
   - for experince over the cloud
   - t3.micro, 2 vCPUs, 1 GiB RAM, 20 GiB root + 2×5 GiB EBS, RHEL 9
  
2. VMware Workstation (Local, 3 VMs)
   - a local solution that mimics the RHCSA test enviroment
   - 2 vCPUs, 2 GiB RAM each, 25 GiB disk each, RHEL 9
  
3. My personal computer
   - a ThinkPad P15s, running Fedora 43 (upstream to RHEL)
   - Special kudos here, I made tons of progress treating my machine like a practice server!
```

## Now for the fun part!


- **Week 1** – Spinning Up the HomeLab → [Read it](./lab-log/week1.md)  
- **Week 2** – Revisiting the Fundamentals → [Read it](./lab-log/week2.md)
  - Bonus - Understanding Persistance → [Read it](./lab-log/week2bonus.md)
- **Week 3** – Automating System Maintenance Tasks → [Read it](./lab-log/week3.md)
- **Week 4** – Troubleshooting the Bootloader → [Read it](./lab-log/week4.md)
- **Week 5** – Processes, Logs, and TuneD → [Read it](./lab-log/week5.md)
- **Week 6** - Configuring Local Storage → [Read it](./lab-log/week6.md)
- **Week 7** - File System Permissions and Disk Quotas → [Read it](./lab-log/week7.md)
- **Week 8** - Creating Filesystems → [Read it](./lab-log/week8.md)
- **Week 9** - Scheduling Tasks with Cron, Anacron, and At → [Read it](./lab-log/week9.md)
- **Week 10** – Linux Startup Processes and Services → [Read it](./lab-log/week10.md)
- **Week 11** – Package Management and Module Streams → [Read it](./lab-log/week11.md)
- **Week 12** – Nework Configuration → [Read it](./lab-log/week12.md)
- **Week 13** – Users and Groups → [Read it](./lab-log/week13.md)
- **Week 14** – SELinux & Firewall Tuning → [Read it](./lab-log/week14.md)
- **Week 15** – Container Management → [Read it](./lab-log/week15.md)

---

> Since you’re down here, have some cute penguins 🐧
> 
> <img src="assets/screenshots/birds-1756510438349-3248.jpg" width="800"/>



