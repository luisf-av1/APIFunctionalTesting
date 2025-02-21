# CI Pipeline For Functional API Testing - Postman & newman 

This repository includes a CI pipeline that automates API testing using Postman (via Newman) and publishes test reports to GitHub Pages. The pipeline runs on:
- **Pushes to the main branch**
- **Pull requests to main**
- **Scheduled execution every weekday at 9 AM UTC**

## Workflow Steps

### 1️⃣ Checkout Code
The pipeline starts by cloning the repository into the runner:
```yaml
- name: ⬇️ Checkout code
  uses: actions/checkout@v3
```

### 2️⃣ Set Up Node.js
It installs Node.js to run Newman:
```yaml
- name: ⚙️ Set up Node.js
  uses: actions/setup-node@v3
  with:
    node-version: '16'
```

### 3️⃣ Install Dependencies
Newman and additional reporters are installed globally:
```yaml
- name: 📦 Install Newman and dependencies
  run: |
    npm install -g newman
    npm install -g newman-reporter-htmlextra
```

### 4️⃣ Prepare Report Storage
A folder is created to store test reports:
```yaml
- name: 📎 Prepare folders to store newman report
  run: mkdir -p newman
```

### 5️⃣ Run API Tests
Newman executes the Postman collection using the provided API key:
```yaml
- name: ▶️ Run Postman Collection through Newman
  run: |
    echo "Go to https://trello.com/u/luisarroyavevalencia/boards" to see live execution
    newman run https://api.postman.com/collections/${{ secrets.POSTMAN_COLLECTION }}?apikey=${{ secrets.POSTMAN_API_KEY }} --folder "Flow" -r cli,junit,htmlextra --reporter-junit-export './newman/report.xml' --reporter-htmlextra-export './newman/report.html'
  env:
    POSTMAN_COLLECTION: ${{ secrets.POSTMAN_COLLECTION }}
    POSTMAN_API_KEY: ${{ secrets.POSTMAN_API_KEY }}
```

### 6️⃣ Save Reports as Artifact
The generated reports are uploaded as artifacts:
```yaml
- name: 📊 Save Test Reports as artifact
  uses: actions/upload-artifact@v4
  with:
    name: trello-api-reports
    path: newman/
```

### 7️⃣ Retrieve Previous Reports
The pipeline fetches the latest reports from the `gh-pages` branch:
```yaml
- name: ➡️ Checkout gh-pages branch
  uses: actions/checkout@v4
  with:
    ref: gh-pages
    path: gh-pages

- name: 📚 Get previous reports from gh-pages branch
  run: |
    rm -rf reports
    mkdir -p reports
    rsync -av --ignore-existing gh-pages/ reports/
```

### 8️⃣ Create a New Report Folder
A timestamped folder is created to store the new report:
```yaml
- name: 🗃 Create folder for current report
  run: |
    export TIMESTAMP=$(date +%Y%m%d-%H%M%S)
    cp -R newman/ "reports/$TIMESTAMP"
    echo "Report generated in: reports/$TIMESTAMP"
```

### 9️⃣ Generate Report Index Page
An `index.html` page is created listing all generated reports:
```yaml
- name: 📝 Generate reports hub page (index.html)
  run: |
    cat << 'EOF' > reports/index.html
    <!DOCTYPE html>
    <html lang="es">
    <head>
      <meta charset="UTF-8">
      <title>Newman Reports</title>
      <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    </head>
    <body>
      <div class="container">
        <h1 class="display-4">Newman Reports</h1>
        <ul>
    EOF
      for dir in reports/*/ ; do
        [ -d "$dir" ] || continue
        dirname=$(basename "$dir")
        if [ "$dirname" != "index.html" ]; then
          echo "<li><a href=\"${dirname}/report.html\">${dirname}</a></li>" >> reports/index.html
        fi
      done
    cat << 'EOF' >> reports/index.html
        </ul>
      </div>
    </body>
    </html>
    EOF
```

### 🔟 Publish Reports to GitHub Pages
The reports are published to the `gh-pages` branch:
```yaml
- name: 🌐 Publish reports in GitHub Pages
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GH_PAT }}
    publish_dir: ./reports
    enable_jekyll: false
```

## Secrets Used
- `POSTMAN_COLLECTION`: The Postman collection ID.
- `POSTMAN_API_KEY`: API key for accessing Postman collections.
- `GH_PAT`: GitHub personal access token for publishing reports.

## Output
- Test reports are stored in the `gh-pages` branch and available via GitHub Pages.
- Reports are archived in timestamped folders.
- `index.html` provides an overview of all test results.

## How to Access Reports
Once the pipeline runs, reports will be available at: https://luisf-av1.github.io/APIFunctionalTesting/

---
🚀 This CI pipeline ensures continuous API testing and easy access to test reports! 🚀

