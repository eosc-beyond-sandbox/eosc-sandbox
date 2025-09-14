# Helpdesk 

## Description of the package (1 paragraph)  

The EOSC Beyond helpdesk solution utilizes Zammad ([https://zammad.com/en](https://zammad.com/en)), an open-source, web-based helpdesk platform. Its source code is accessible on GitHub and maintained by the Zammad Foundation. Additionally, the helpdesk can be seamlessly integrated with the EOSC Beyond AAI service.


## Code  

Zammad GitHub repository is located at:
[https://github.com/zammad/zammad](https://github.com/zammad/zammad)

On Ubuntu/Debian, OpenSUSE/SLES, and CentOS/RHEL, it is also available through the package manager, which we have tested and recommend for deployment.
## Dependencies  

The detailed Zammad hardware requirements can be found:
[https://docs.zammad.org/en/latest/prerequisites/hardware.html](https://docs.zammad.org/en/latest/prerequisites/hardware.html)

The detailed Zammad software requirements can be found:
[https://docs.zammad.org/en/latest/prerequisites/software.html](https://docs.zammad.org/en/latest/prerequisites/software.html)

## Database setup  

## Configuration files and examples  

## Deployment gidelines  

We tested Zammad deployment on a Debian 12 server using Zammad packages from the package manager system. However, Zammad can also be deployed via:
- Docker, [https://docs.zammad.org/en/latest/install/docker-compose.html](https://docs.zammad.org/en/latest/install/docker-compose.html) 
    
- Kubernetes, [https://docs.zammad.org/en/latest/install/kubernetes.html](https://docs.zammad.org/en/latest/install/kubernetes.html) 
    
- From source, [https://docs.zammad.org/en/latest/install/source.html](https://docs.zammad.org/en/latest/install/source.html)  
    
Detailed deployment instructions for Debian 12 OS are available at [https://docs.zammad.org/en/latest/install/package.html](https://docs.zammad.org/en/latest/install/package.html)  and include setting up APT repositories and installing Zammad services. The guide also covers recommended firewall setup (using ufw or nftables) and SELinux configuration.

## Documentation  

## Verification  
 
## Licensing  

Zammad is licensed under the GNU AGPLv, which requires sharing source code even if users only access the software over a network. Make sure users can view, modify, and redistribute the source code, including any modifications.

