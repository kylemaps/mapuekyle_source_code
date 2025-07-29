# Kyle Mapue's Personal Portfolio Website

This repository contains the source code for Kyle Mapue's personal portfolio website, built using the Hugo static site generator and the PaperMod theme.

## Project Structure

- `config.yaml`: Main Hugo site configuration.
- `content/`: Markdown files for blog posts and other pages.
- `static/`: Static assets like images (including blog post images) and favicons.
- `themes/PaperMod/`: The PaperMod Hugo theme, included as a Git submodule.

## Local Development

To run this site locally, you need to have Hugo installed.

1.  **Clone the repository:**
    ```bash
    git clone <your-repository-url>
    cd source_code_mapuekyle
    ```
    (Replace `<your-repository-url>` with the actual URL of your GitHub repository.)

2.  **Initialize and update Hugo submodules:**
    The PaperMod theme is included as a Git submodule.
    ```bash
    git submodule update --init --recursive
    ```

3.  **Run the Hugo development server:**
    ```bash
    hugo serve
    ```
    Your site should now be accessible at `http://localhost:1313` (or another port specified by Hugo).

## Deployment

This site is deployed as a static site to Netlify. Continuous deployment is configured via Git integration with Netlify, where Netlify automatically builds and deploys the site on every push to the `main` branch.

This site used to be run on a private virtual server via Digital Ocean's droplet for years but for economical reasons it is in Netlify, for now. Ideally to be put again in VPS along with other projects in the future (using docker compose or k3d)

## Blog Posts

-   `my-kubernetes-journey.md`: Details a Kubernetes project, including manual deployment, service connectivity, GitOps automation with ArgoCD, and reliability enhancements.
-   `my-todo-app-journey.md`: Chronicles the journey of building a Todo app, covering microservices, real-world failures, Kustomize, CI/CD with GitHub Actions, monitoring with Prometheus/Grafana, and advanced deployment strategies.
-   `devops-journey.md`: Explains the story behind building this very website, touching on IaC (Terraform), configuration management (Ansible), and CI/CD (GitHub Actions).
-   `devops-with-docker.md`: A learning log detailing a journey through Docker fundamentals, Docker Compose, and advanced Docker deployment patterns.