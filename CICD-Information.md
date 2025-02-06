# Jenkins CI/CD Setup Guide

## **Jenkins Overview**

- Jenkins is **free** and has **powerful plugins**.
- It helps automate **CI/CD pipelines**.

## **Jenkins Server Access**

- URL: `http://34.254.6.118/:8080/`
- Username: **devopslondon**
- Password: **Confidential**

---

## **Setting Up Your First Jenkins Job**

1. Create a **New Item** (Freestyle Project).
2. Name it **nadia-first-project** → Click **OK**.
3. Add a **description** (e.g., Testing Jenkins).
4. Enable **Discard Old Builds** → Set max builds to **5**.
5. **Add Build Step** → Select **Execute Shell** → Enter:
   ```sh
   uname -a
   ```
6. Save & Click **Build Now**.
7. Check the **console output** to see the result.

---

## **Creating a Simple Pipeline to Show Date & Time**

1. Create a **New Item** (Freestyle Project).
2. Add **Build Step** → **Execute Shell**:
   ```sh
   date
   ```
3. Save & Click **Build Now**.
4. Check the **console output**.

---

## **Creating a Multi-Stage Pipeline**

1. Open your **first project job**.
2. **Post-Build Actions**:
   - Select **Build Other Projects**.
   - Enter the name of the **next project**.
   - Choose **Trigger only if build is stable**.
3. Save & Click **Build Now**.
4. Verify the logs in **console output**.

---

## **Setting Up a CI/CD Pipeline for Sparta Test App**

### **Workflow**:

1. **Developer pushes code** to GitHub (**Dev Branch**).
2. **GitHub triggers Jenkins** via Webhook.
3. **Jenkins executes CI/CD pipeline**:
   - **Job 1**: Runs tests.
   - **Job 2**: Merges **dev → main** if tests pass.
   - **Job 3**: Deploys code to AWS EC2.

### **Jenkins Components**:

- **Master Node**: Manages jobs.
- **Agent Nodes**: Run tasks like testing & deployment.

---

## **Setting Up GitHub & Local Repository**

1. Create a **new GitHub repo**:
   - Name: `sparta-test-app-cicd`
   - Set to **Public**.
2. In your local system:
   ```sh
   cd ~/Documents/github
   mkdir sparta-test-app-cicd
   cd sparta-test-app-cicd
   git init
   git remote add origin https://github.com/Nmudassar/sparta-test-app-cicd.git
   ```
3. Push code to GitHub:
   ```sh
   git add .
   git commit -m "Initial commit"
   git push -u origin main
   ```

---

## **Adding SSH Credentials to Jenkins**

### **Generate SSH Key (Local Machine)**:

```sh
ssh-keygen -t rsa -b 4096 -C "nadia.syed77@gmail.com"
```

- Name the key: `nadia-jenkins-github-key`

### **Add Public Key to GitHub**:

1. Go to **GitHub → Repo → Settings → Deploy Keys**.
2. Click **Add Key** → Name it `nadia-jenkins-github-key`.
3. Run in terminal:
   ```sh
   cat ~/.ssh/nadia-jenkins-github-key.pub
   ```
4. Copy the output & paste it in GitHub.
5. Check **Allow write access** → Save.

---

## **Setting Up Jenkins Job 1 (CI - Test Code)**

1. Open **Jenkins** → Click **New Item**.
2. Name it **nadia-job1-ci-test** → Select **Freestyle Project**.
3. Enable **Discard Old Builds** (Max 5).
4. **Source Code Management** → Select **Git**:
   - URL: `git@github.com:Nmudassar/sparta-test-app-cicd.git`
   - Credentials: Select `nadia-jenkins-github-key`.
5. **Branches to Build** → `*/main`
6. **Build Steps**:
   ```sh
   cd app
   npm install
   npm test
   ```
7. **Save & Build Now**.

---

## **Setting Up GitHub Webhook**

1. In Jenkins Job **Configuration**:
   - Change **Branch Specifier** to `*/dev`.
   - Enable **GitHub hook trigger for GITScm polling**.
2. In **GitHub → Repo → Settings → Webhooks**:
   - Add Webhook:
     - **Payload URL**: `http://34.254.6.118/:8080/github-webhook/`
     - **Disable SSL verification** → Save.

### **Test Webhook**:

```sh
nano README.md  # Make a small change
git add .
git commit -m "Testing webhook"
git push origin dev
```

- Jenkins should **automatically trigger the build**.

---

## **Creating a Dev Branch Locally**

```sh
git checkout -b dev
nano README.md  # Make a change
git add .
git commit -m "dev branch test"
git push origin dev
```

- The Jenkins job should **automatically run**.

---

## **Setting Up Job 2 (Merge Dev to Main)**

1. **Create a New Job** → Name: `nadia-job2-ci-merge`.
2. **Trigger Job 2 only if Job 1 is successful**:
   - Enable **Build after other projects are built**.
   - Enter **Job 1 name** (`nadia-job1-ci-test`).
3. **Source Code Management**:
   - **Git URL**: `git@github.com:Nmudassar/sparta-test-app-cicd.git`
   - Select **SSH Credentials** (`nadia-jenkins-github-key`).
4. **Build Step - Execute Shell**:
   ```sh
   git checkout main
   git pull origin main
   git merge --ff-only origin/dev
   git push origin main
   ```
5. Save & Run the Job.

---

## **Final Notes**

- Webhook should trigger Jenkins when code is pushed.
- Job 1 runs tests on `dev`.
- Job 2 merges `dev` into `main` if tests pass.
- Job 3 (deployment) can now be set up to deploy to EC2
