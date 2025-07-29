---
title: "The Story of This Very Website"
date: "2024-11-11"
author: "Kyle Mapue"
categories:
  - "devops"
  - "container"
  - "docker"
---

# Introduction

As a passionate individual eager to delve into the world of DevOps, I found myself facing a common question: "What is a good beginner project to gain practical experience?" After exploring various resources and communities, I stumbled upon an insightful blog post titled "The Best DevOps Project for a Beginner" by Logan Marchione. Inspired by his guidance, I embarked on an exciting journey to build my own static website, and I would like to share my personal experience and lessons learned along the way.

> Often it is difficult to get hands-on experience in DevOps. Doing a beginner challenge could solidfy otherwise only theoretical concepts.

# Static site

He recommended building a static site. *That's the answer*.

![meme](/blog/devops-journey/static.gif)

## Why

A static site is a great beginner project because:
- It gave me a great chance to finally purchase `mapuekyle.com`
- I touched on how websites works (e.g., domain names, DNS, web hosting, certificates)
- Experienced more [infrastructure-as-code](https://en.wikipedia.org/wiki/Infrastructure_as_code) (IaC) with [Terraform](https://www.terraform.io/)
- I learned [configuration management](https://en.wikipedia.org/wiki/Software_configuration_management) with [Ansible](https://www.ansible.com/)
- I got to check how a [static site generator](https://en.wikipedia.org/wiki/Static_site_generator) works.
- Touches on how [Git](https://git-scm.com/) works
- Messed with [continuous integration and continuous delivery](https://en.wikipedia.org/wiki/CI/CD) (CI/CD) with [GitHub Actions](https://github.com/features/actions)

## How

Below are the steps I took based on the outline of the steps for the challenge.

1. Purchase a domain
    - I took the liberty to browse through different sites/marketplace:
    - [Hover](https://www.hover.com/), [Cloudflare](https://www.cloudflare.com/products/registrar/), [Namecheap](https://www.namecheap.com/), [AWS](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html), etc...
    - Given that this is a personal website, I opted for a '.com' on Namecheap
1. Create a two Github repositories
    - One for IaC code (Terraform and Ansible) [repo-iac](https://github.com/kylemaps/beginner-devops-project-IaC)
    - One for static site's code [repo-static-site](https://www.cloudflare.com/products/registrar/)
1. Provision a [virtual private server](https://en.wikipedia.org/wiki/Virtual_private_server) (VPS) in the cloud using Terraform
    - So I looked up the recommended options for VPS that has Terraform support: ([DigitalOcean](https://registry.terraform.io/providers/digitalocean/digitalocean/), [AWS](https://registry.terraform.io/providers/hashicorp/aws/), [Linode](https://registry.terraform.io/providers/linode/linode/), [OVH](https://registry.terraform.io/providers/ovh/ovh/), [Oracle Cloud](https://registry.terraform.io/providers/oracle/oci/), [Scaleway](https://registry.terraform.io/providers/scaleway/scaleway/), etc...)
    - For this project I used DigitalOcean (I like it because of the straightforward pricing)
    - This code was checked out on its respective repo
    - Notes: Don't hard-code your API keys anywhere in your Terraform code, more like NEVER hard-code them.
    - Notes: There is an option for me to store them on my DigitalOcean account but they can also be stored [remotely](https://www.terraform.io/language/state/remote) (HashiCorp offers free state storage in Terraform Cloud)
    - Improvements for future: Use [Atlantis](https://www.runatlantis.io/) with your GitHub account to run Terraform via pull requests to GitHub
1. Setup DNS using Terraform
    - After I got the IP of my VPS, I linked `mapuekyle.com` to `<my vps ip>`
    - You can use a separate DNS provider (like [Cloudflare](https://registry.terraform.io/providers/cloudflare/cloudflare/), [NS1](https://registry.terraform.io/providers/ns1-terraform/ns1/), or [DNSimple](https://registry.terraform.io/providers/dnsimple/dnsimple/) ), but your VPS provider might also offer DNS (e.g., [DigitalOcean](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/record), [AWS Route53](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record), [Linode](https://registry.terraform.io/providers/linode/linode/latest/docs/resources/domain_record), [Scaleway](https://registry.terraform.io/providers/scaleway/scaleway/latest/docs/resources/domain_record), etc...)
    - This code was checked into Git on GitHub
1. Configure the VPS using Ansible
    - After the VPS is online, I installed updates, setup a user, install packages, mess with configuration files, etc...
    - I also installed a webserver `Nginx`.
    - This code was checked into Git on GitHub
    - Notes: Learned that I can get a TLS certificate for free from [Let's Encrypt](https://letsencrypt.org/) and configure your webserver to redirect from port 80 to 443
    - Improvements for future: Instead of making one big playbook, use [roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)
1. Run Ansible through Docker
    - I thought it would be a good practice for me to run my Ansible playbook through Docker (but you can definitely run it as usual)
    - I created a `dockerfile` inside my Ansible directory and installed respective dependencies for me to run the playbook.
1. Create the static site locally on your PC
     - I choose from an extensive list of static site generator ([here](https://jamstack.org/generators/))
     - I initially used a different site that runs on react but opted for~
     - [Hugo](https://github.com/gohugoio/hugo),(consider things like speed, available themes, the language the templates are written in, plugins, if you're migrating from another data source, etc...)
     - I chose Hugo for its simplicity and speed.
     - This code was checked into Git on Github
1. Deploy the site to your VPS using Setup GitHub Actions
    - The goal is to automate deploying the static site's rendered code from GitHub to VPS
    - The automation is set on each push but you can definitely add other triggers (e.g., on each commit to Git, on a schedule, on a tag, etc...)
    - Improvements for the future: Use GitHub Actions to lint your Terraform code with [tflint](https://github.com/terraform-linters/tflint) and Ansible code with [ansible-lint](https://github.com/ansible/ansible-lint)
    - Bonus: Setup a free [GitHub Pages](https://pages.github.com/) domain at `mapuekyle.github.io` and push a dev/test version of your site to there but you can also do it on [Netlify](https://www.netlify.com/pricing/).

# Cost

In my setup, I'm spending $84/year on my site and the surrounding infrastructure.

| Product              | Cost (per year)     |
|----------------------|---------------------|
| Domain (NameCheap)   | $12                 |
| DNS (Cloudflare)     | Free                |
| VPS (DigitalOcean)   | $72                 |


# Conclusion

In conclusion, building a static website as a beginner project proved to be an excellent choice. It laid the foundation for my DevOps aspirations and provided a solid platform for future growth. I encourage fellow beginners to embark on this journey, embrace the challenges, and experience the transformative power of hands-on learning in DevOps.

I am looking forward to doing [The Cloud Resume Challenge](https://cloudresumechallenge.dev/) and navigating my way through [roadmap.sh](https://roadmap.sh/devops)).

\-Kyle