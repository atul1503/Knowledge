Notes:

1. **GitHub Actions is like having a robot assistant for your code.** It automatically runs tasks when things happen to your repository - like testing your code when you push changes, or deploying your app when you merge to main.

2. **Workflows are the instruction sets for your robot.** They live in `.github/workflows/` folder and are written in YAML. Each workflow file describes what to do and when to do it.
   ```yaml
   name: CI Pipeline                    # What to call this workflow (shows up in GitHub UI)
   
   on:                                  # When to run this workflow (the trigger events)
     push:                              # Run when code is pushed
       branches: [ main, develop ]      # Only on these branches
     pull_request:                      # Also run on pull requests
       branches: [ main ]               # Only PRs targeting main branch
   ```

3. **Jobs are the main tasks in your workflow.** Each job runs on a separate virtual machine (called a runner). Jobs run in parallel by default unless you tell them to wait for each other.
   ```yaml
   jobs:                                # List of jobs to run
     test:                              # Job name (you choose this)
       runs-on: ubuntu-latest           # Which operating system to use (ubuntu-latest, windows-latest, macos-latest)
       
       steps:                           # List of steps to run in this job
         - name: Checkout code          # Human-readable name for this step
           uses: actions/checkout@v4    # Use a pre-built action to download your code
           
         - name: Run tests              # Another step
           run: npm test                # Shell command to execute
   ```

4. **Steps are the individual commands within a job.** They run one after another. If any step fails, the whole job fails (unless you tell it otherwise).
   ```yaml
   steps:
     - name: Setup Node.js              # Step to install Node.js
       uses: actions/setup-node@v4      # Pre-built action for setting up Node
       with:                            # Parameters to pass to this action
         node-version: '18'             # Which version of Node.js to install
         
     - name: Install dependencies       # Step to install npm packages
       run: npm install                # Shell command to run
       
     - name: Build project             # Step to build the project
       run: npm run build              # Another shell command
   ```

5. **Actions are reusable pieces of code.** Think of them like functions - someone else wrote them, you just use them. The `uses` keyword tells GitHub to run an action, while `run` executes shell commands directly.
   ```yaml
   steps:
     - uses: actions/checkout@v4        # Use someone else's action (downloads your code)
     - uses: actions/setup-python@v4    # Use action to install Python
       with:                            # Pass parameters to the action
         python-version: '3.9'
         
     - run: pip install -r requirements.txt    # Run your own shell command
     - run: python -m pytest                   # Run another shell command
   ```

6. **Environment variables let you store values that steps can use.** Define them at workflow, job, or step level. Use `${{ env.VARIABLE_NAME }}` to access them.
   ```yaml
   env:                                 # Workflow-level environment variables
     NODE_VERSION: '18'
     
   jobs:
     build:
       runs-on: ubuntu-latest
       env:                             # Job-level environment variables
         DATABASE_URL: 'postgres://localhost:5432/testdb'
         
       steps:
         - name: Build app
           env:                         # Step-level environment variables
             API_KEY: 'test-key-123'
           run: |
             echo "Node version: ${{ env.NODE_VERSION }}"
             echo "Database: ${{ env.DATABASE_URL }}"
             npm run build
   ```

7. **Secrets store sensitive information safely.** Never put passwords or API keys directly in your workflow files. Store them in repository settings and access with `${{ secrets.SECRET_NAME }}`.
   ```yaml
   steps:
     - name: Deploy to production
       env:
         API_KEY: ${{ secrets.PRODUCTION_API_KEY }}    # Get secret from repository settings
         DB_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}  # Another secret
       run: |
         echo "Deploying with API key..."
         ./deploy.sh
   ```

8. **Conditions control when steps or jobs run.** Use `if` to skip things based on branch, event type, or previous step results.
   ```yaml
   jobs:
     deploy:
       runs-on: ubuntu-latest
       if: github.ref == 'refs/heads/main'    # Only run on main branch
       
       steps:
         - name: Deploy to staging
           if: github.event_name == 'pull_request'    # Only on pull requests
           run: ./deploy-staging.sh
           
         - name: Deploy to production
           if: github.ref == 'refs/heads/main' && github.event_name == 'push'    # Only on pushes to main
           run: ./deploy-production.sh
   ```

9. **Matrix builds let you test against multiple versions or environments.** GitHub will create separate jobs for each combination.
   ```yaml
   jobs:
     test:
       runs-on: ubuntu-latest
       strategy:                        # Strategy for running multiple variations
         matrix:                        # Matrix defines the combinations to test
           node-version: [16, 18, 20]   # Test against these Node.js versions
           os: [ubuntu-latest, windows-latest]    # Test on these operating systems
           
       steps:
         - uses: actions/checkout@v4
         - uses: actions/setup-node@v4
           with:
             node-version: ${{ matrix.node-version }}    # Use the current matrix value
         - run: npm test
   ```

10. **Artifacts let you save files between jobs or download them later.** Useful for saving build outputs, test reports, or logs.
    ```yaml
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - run: npm run build
          
          - name: Upload build artifacts    # Save files for later use
            uses: actions/upload-artifact@v4
            with:
              name: build-files             # Name for this artifact
              path: dist/                   # Which files/folders to save
              
      deploy:
        needs: build                        # Wait for build job to finish
        runs-on: ubuntu-latest
        steps:
          - name: Download build artifacts  # Get files from previous job
            uses: actions/download-artifact@v4
            with:
              name: build-files             # Which artifact to download
              path: ./dist                  # Where to put the files
          - run: ./deploy.sh
    ```

11. **Job dependencies control the order jobs run.** Use `needs` to make jobs wait for others to complete successfully.
    ```yaml
    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - run: npm test
          
      build:
        needs: test                         # Wait for test job to succeed
        runs-on: ubuntu-latest
        steps:
          - run: npm run build
          
      deploy:
        needs: [test, build]                # Wait for both test AND build to succeed
        runs-on: ubuntu-latest
        steps:
          - run: ./deploy.sh
    ```

12. **Outputs let jobs share data with each other.** One job can calculate something and pass it to jobs that depend on it.
    ```yaml
    jobs:
      setup:
        runs-on: ubuntu-latest
        outputs:                            # Define outputs this job produces
          version: ${{ steps.get_version.outputs.version }}    # Output named 'version'
          
        steps:
          - uses: actions/checkout@v4
          - name: Get version
            id: get_version                 # Give this step an ID so we can reference it
            run: |
              VERSION=$(cat package.json | jq -r '.version')
              echo "version=$VERSION" >> $GITHUB_OUTPUT    # Set the output value
              
      deploy:
        needs: setup
        runs-on: ubuntu-latest
        steps:
          - name: Deploy version
            run: |
              echo "Deploying version ${{ needs.setup.outputs.version }}"    # Use output from setup job
              ./deploy.sh ${{ needs.setup.outputs.version }}
    ```

13. **Caching speeds up workflows by saving dependencies between runs.** Cache things like npm packages, pip packages, or build outputs.
    ```yaml
    steps:
      - uses: actions/checkout@v4
      
      - name: Cache Node modules         # Try to restore cached dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm                    # Where the cache files are stored
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}    # Unique key based on OS and lock file
          restore-keys: |                 # Fallback keys if exact match not found
            ${{ runner.os }}-node-
            
      - run: npm install                 # This will be faster if cache hit
      - run: npm test
    ```

14. **Workflow dispatch lets you manually trigger workflows.** Add a button in GitHub UI to run workflows on demand with custom inputs.
    ```yaml
    on:
      workflow_dispatch:                  # Allow manual triggering
        inputs:                           # Define input parameters for manual runs
          environment:                    # Input name
            description: 'Environment to deploy to'    # Description shown in UI
            required: true                # Whether this input is required
            default: 'staging'            # Default value
            type: choice                  # Input type (choice, string, boolean, number)
            options:                      # Available options for choice type
              - staging
              - production
              
    jobs:
      deploy:
        runs-on: ubuntu-latest
        steps:
          - name: Deploy to environment
            run: |
              echo "Deploying to ${{ github.event.inputs.environment }}"    # Use the input value
              ./deploy.sh ${{ github.event.inputs.environment }}
    ```

15. **Reusable workflows let you share common workflows across repositories.** Create a workflow once and call it from other workflows.
    ```yaml
    # In .github/workflows/reusable-deploy.yml
    on:
      workflow_call:                      # This workflow can be called by others
        inputs:                           # Inputs that calling workflows must provide
          environment:
            required: true
            type: string
        secrets:                          # Secrets that calling workflows must provide
          api_key:
            required: true
            
    jobs:
      deploy:
        runs-on: ubuntu-latest
        steps:
          - name: Deploy
            env:
              API_KEY: ${{ secrets.api_key }}
            run: ./deploy.sh ${{ inputs.environment }}
    ```
    
    ```yaml
    # In another workflow that uses the reusable one
    jobs:
      call-deploy:
        uses: ./.github/workflows/reusable-deploy.yml    # Path to reusable workflow
        with:                             # Pass inputs to reusable workflow
          environment: production
        secrets:                          # Pass secrets to reusable workflow
          api_key: ${{ secrets.PROD_API_KEY }}
    ```

16. **Common workflow patterns for different types of projects:**

    **Node.js/JavaScript project:**
    ```yaml
    name: Node.js CI
    on: [push, pull_request]
    
    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - uses: actions/setup-node@v4
            with:
              node-version: '18'
              cache: 'npm'                # Automatically cache npm dependencies
          - run: npm ci                   # Install exact versions from package-lock.json
          - run: npm run lint             # Run linting
          - run: npm test                 # Run tests
          - run: npm run build            # Build the project
    ```

    **Python project:**
    ```yaml
    name: Python CI
    on: [push, pull_request]
    
    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - uses: actions/setup-python@v4
            with:
              python-version: '3.9'
          - name: Install dependencies
            run: |
              python -m pip install --upgrade pip
              pip install -r requirements.txt
          - run: python -m pytest        # Run tests with pytest
          - run: python -m flake8        # Run linting
    ```

    **Docker build and push:**
    ```yaml
    name: Docker Build
    on:
      push:
        branches: [main]
        
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          
          - name: Login to Docker Hub
            uses: docker/login-action@v2
            with:
              username: ${{ secrets.DOCKER_USERNAME }}
              password: ${{ secrets.DOCKER_PASSWORD }}
              
          - name: Build and push
            uses: docker/build-push-action@v4
            with:
              context: .                  # Build context (where Dockerfile is)
              push: true                  # Push to registry after building
              tags: myapp:latest          # Tag for the built image
    ```

17. **Debugging workflows when things go wrong:**
    ```yaml
    steps:
      - name: Debug information
        run: |
          echo "Event: ${{ github.event_name }}"
          echo "Ref: ${{ github.ref }}"
          echo "SHA: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
          env                             # Print all environment variables
          
      - name: Enable debug logging       # Add this to get more detailed logs
        run: echo "::debug::This is a debug message"
        
      - name: Continue on error          # Don't fail the job if this step fails
        run: exit 1                      # This command will fail
        continue-on-error: true          # But the job will continue anyway
    ```

18. **Useful GitHub context variables you can use in workflows:**
    ```yaml
    steps:
      - name: Show context info
        run: |
          echo "Repository: ${{ github.repository }}"           # owner/repo-name
          echo "Branch: ${{ github.ref_name }}"                 # branch name
          echo "Commit SHA: ${{ github.sha }}"                  # full commit hash
          echo "Short SHA: ${{ github.sha }}"                   # first 7 chars of hash
          echo "Event: ${{ github.event_name }}"                # push, pull_request, etc.
          echo "Actor: ${{ github.actor }}"                     # who triggered this
          echo "Workflow: ${{ github.workflow }}"               # workflow name
          echo "Job: ${{ github.job }}"                         # current job name
          echo "Run ID: ${{ github.run_id }}"                   # unique run identifier
    ``` 