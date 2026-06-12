React App
Create workflow file .github/workflows/deploy.yml:
yamlname: Deploy to SFTP Server

on:
  push:
    branches: [ main ]  # Deploy on push to main
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # Checkout code
      - uses: actions/checkout@v3

      # Setup Node.js
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      # Install dependencies
      - name: Install dependencies
        run: npm install

      # Build React app
      - name: Build React app
        run: npm run build

      # Deploy to SFTP
      - name: Deploy to SFTP
        uses: presshold/sftp-deploy@v1.3.0
        with:
          host: ${{ secrets.SFTP_SERVER }}
          port: ${{ secrets.SFTP_PORT }}
          username: ${{ secrets.SFTP_USERNAME }}
          password: ${{ secrets.SFTP_PASSWORD }}
          local_path: './dist/*'
          remote_path: '/public_html/'
          delete: true
3. Alternative: Using lftp (more control):
yamlname: Deploy to SFTP Server

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install and Build
        run: |
          npm install
          npm run build

      - name: Deploy via SFTP
        run: |
          apt-get update && apt-get install -y lftp
          lftp -e "
          set sftp:auto-confirm yes
          set ssl:verify-certificate no
          open -u ${{ secrets.SFTP_USERNAME }},${{ secrets.SFTP_PASSWORD }} -p ${{ secrets.SFTP_PORT }} ${{ secrets.SFTP_SERVER }}
          mirror -R dist /public_html
          quit
          "
