

# **Checkers: A Self-Sustaining Uptime Monitor**

This document outlines the mission, features, and technical architecture of Checkers, a specialized uptime monitoring service designed to operate entirely within the free tier of modern cloud platforms.

### **1. Our Mission: What is Checkers?**

Checkers is a simple, automated service that monitors the health of a website by pinging it at regular intervals. Its primary purpose is to provide a reliable, "set it and forget it" uptime monitoring solution that is completely free to operate. It is built for developers, hobbyists, and small teams who need to ensure their web services remain responsive and "awake" without incurring the cost of traditional monitoring or database services.

### **2. The Problem: Why Does Checkers Exist?**

Many modern hosting platforms (like Vercel, Render, and Hugging Face Spaces) offer generous free tiers. However, to conserve resources, these platforms often put services "to sleep" after a period of inactivity. When a user finally visits the sleeping service, they experience a long cold start, which can last anywhere from a few seconds to nearly a minute.

This creates a poor user experience and can cause issues for background services that need to be consistently available. Checkers solves this problem by acting as a persistent, friendly visitor, pinging a target website every 15 minutes to keep it active and responsive 24/7.

### **3. User Features: Your Personal Watchdog**

The user experience is designed to be minimalist and effective. Any user can access the public Checkers instance and benefit from the following features:

*   **Add a Site:** Users can submit a single URL to the monitoring list. The system is designed with a "one site per person" philosophy in mind, though this is not strictly enforced by user accounts in the current version.
*   **Live Dashboard:** The main page provides a clean, at-a-glance view of all monitored sites.
*   **Status Indicators:** Each site clearly displays its current status (e.g., `OK`, `FAIL`, `ERROR`) based on the last check.
*   **Recent Ping History:** Users can see a short history of the last few pings for their site, including the timestamp, status code, and response time (latency).
*   **24/7 Background Monitoring:** Once a site is added, Checkers takes over. A dedicated background process ensures the site is pinged every 15 minutes, indefinitely, with no further user interaction required.

### **4. Admin Features: The Control Panel**

For the system administrator, Checkers provides a set of secure, powerful tools to manage and observe the service:

*   **System Health Overview:** A conceptual dashboard to see total sites monitored and the status of background workers.
*   **Complete Site List:** View all URLs currently being monitored by the system.
*   **Manual Backup/Restore (Implicit):** While backups are automated, the administrator has the tools to manually trigger or verify the backup process.
*   **Secure Database Access:** A private, token-protected endpoint (`/admin/download_db`) allows the administrator to securely download a live snapshot of the application's entire database (`app.db`). This is crucial for debugging, migration, or manual analysis.
*   **Private by Default:** All administrative functions and API documentation endpoints (`/docs`, `/redoc`) are disabled to prevent public discovery and ensure the system's attack surface is minimized.

### **5. The Technical Blueprint**

Checkers is built on a modern, efficient technology stack chosen specifically for its performance and compatibility with free-tier hosting environments.

*   **Backend Framework:** **FastAPI** provides a high-performance web server and API, running on **Uvicorn**.
*   **Frontend:** A simple HTML interface rendered with the **Jinja2** templating engine and styled with **Tailwind CSS** for a clean, responsive look.
*   **Database:** **SQLite**. We intentionally chose SQLite because it's a serverless, single-file database. This file, `app.db`, is the "soul" of our application—it contains all sites and ping history. Crucially, it runs from the ephemeral `/tmp` directory, which is wiped clean every time the service restarts.
*   **Monitoring Engine:** The core of Checkers is its **Supervisor/Worker architecture**.
    *   A central **Supervisor** process runs within the main application. Its job is to read the list of active sites from the database.
    *   For each active site, the Supervisor spawns a dedicated, isolated **Worker** process using Python's `subprocess` module.
    *   Each Worker is an independent, lightweight script that runs in an infinite loop, responsible for pinging its assigned URL every 15 minutes and recording the result back to the SQLite database.
    *   This model ensures that the failure of one worker does not affect any others and that monitoring is truly parallel and non-blocking.

### **6. The "Free Tier" Philosophy: Our Core Innovation**

The most critical challenge was achieving data persistence without a paid, persistent disk or a managed database. Checkers overcomes this with a novel backup and restore workflow that leverages the free resources of the Hugging Face Hub.

**This is our approach to avoid all costs:**

1.  **Embrace Ephemeral Storage:** We accept that the live database at `/tmp/app.db` is temporary. It's fast and efficient for runtime operations, but we treat it as disposable.

2.  **Use a Dataset Repo as a "Cloud Hard Drive":** We create a **private Hugging Face Dataset repository**. This service is designed for storing large datasets but works perfectly as a free, versioned, private file storage system. This repository becomes our permanent "hard drive" for the `app.db` file.

3.  **Restore on Startup:** When the Hugging Face Space starts (or restarts after a crash or sleep), its very first action is to securely connect to the private Dataset repo and **download the latest `app.db` backup** into its `/tmp` directory. This single action instantly restores the application to its last known state. If no backup exists, it creates a fresh, empty database.

4.  **Automated Backup During Runtime:** A background task runs every few hours. It takes the current `app.db` file from `/tmp` and **uploads it to the private Dataset repo**, overwriting the previous backup. This ensures we always have a recent snapshot of the system's state stored safely and permanently.

**The result is a self-healing, resilient system. Even if the entire Space is rebuilt or crashes, it will always revive itself using the latest snapshot from its free, permanent storage, costing zero dollars.**

### **7. Our Motive**

The motive behind Checkers is to demonstrate that powerful, reliable, and "always-on" services can be built and deployed without financial cost. It serves as a practical blueprint for developers to create resilient applications by creatively combining the features of free-tier platforms. We believe in empowering the developer community with tools and techniques that lower the barrier to building and sharing useful software.

Checkers is more than a utility; it's a statement about sustainable, cost-effective engineering in the modern cloud era.
