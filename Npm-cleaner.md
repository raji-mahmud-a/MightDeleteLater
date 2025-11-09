# **npm-cleaner** - Find and Remove Unused NPM Packages

## **The Problem (Super Simple)**

You install packages to try them out:

```bash
npm install moment
npm install axios
npm install lodash
npm install chalk
npm install uuid
# ...50 more packages over 6 months
```

**6 months later:**
- Your `node_modules` folder is 500MB
- Your `package.json` has 60 dependencies
- You forgot what half of them do
- **BUT you only actually USE 30 of them**

**The other 30 packages?**
- Wasting disk space
- Slowing down `npm install`
- Adding security vulnerabilities
- Costing money (deploy size, CDN bandwidth)
- Making your project look messy

---

## **The Solution**

```bash
npx npm-cleaner

# Output:
🔍 Scanning your project...

Found 60 packages in package.json
Analyzing 234 JavaScript files...

✅ USED (30 packages)
   These are imported in your code

⚠️  UNUSED (25 packages) 
   These are NEVER imported anywhere!
   
   1. moment (2.9 MB) - Last used: Never
   2. axios (1.2 MB) - Last used: Never  
   3. request (5.1 MB) - Last used: 6 months ago
   4. chalk (0.5 MB) - Last used: Never
   5. uuid (0.3 MB) - Last used: Never
   ...20 more

🤔 MAYBE UNUSED (5 packages)
   Can't determine if used (might be dynamic imports)
   
   1. dotenv
   2. cors
   ...3 more

💾 Total waste: 47.3 MB
💰 Potential savings: $5.40/month in CDN costs

Remove unused packages? [y/N]
```

If you type `y`:

```bash
✓ Removing moment...
✓ Removing axios...
✓ Removing request...
✓ Removing chalk...
✓ Removing uuid...

✅ Removed 25 packages!
✅ Freed 47.3 MB of disk space
✅ Your package.json is now clean

Run 'npm install' to update node_modules
```

---

## **How It Works (Simple)**

### **Step 1: Read package.json**
```javascript
const packageJson = require('./package.json');
const allPackages = Object.keys(packageJson.dependencies);
// Example: ['express', 'moment', 'axios', 'lodash', ...]
```

### **Step 2: Scan all your code files**
```javascript
const files = findAllJavaScriptFiles('./src');
// Finds: src/server.js, src/routes/api.js, src/utils/helpers.js, ...

const imports = [];
files.forEach(file => {
  const code = readFile(file);
  
  // Find: require('express')
  const requires = code.match(/require\(['"](.+?)['"]\)/g);
  
  // Find: import express from 'express'
  const importStatements = code.match(/import .+ from ['"](.+?)['"]/g);
  
  imports.push(...requires, ...importStatements);
});
```

### **Step 3: Compare**
```javascript
const usedPackages = [];
const unusedPackages = [];

allPackages.forEach(pkg => {
  if (imports.includes(pkg)) {
    usedPackages.push(pkg);
  } else {
    unusedPackages.push(pkg);
  }
});

console.log('Unused:', unusedPackages);
```

### **Step 4: Remove**
```javascript
if (userConfirms) {
  unusedPackages.forEach(pkg => {
    exec(`npm uninstall ${pkg}`);
  });
}
```

**That's it!** Simple 4-step process.

---

## **Features (All Simple)**

### **1. Safe Mode (default)**
```bash
npx npm-cleaner

# Shows what WOULD be removed, but doesn't remove anything
# User must confirm before removal
```

### **2. Auto Mode**
```bash
npx npm-cleaner --auto

# Automatically removes unused packages
# (for CI/CD or brave users)
```

### **3. Dry Run**
```bash
npx npm-cleaner --dry-run

# Just shows results, never removes anything
# Perfect for checking before cleanup
```

### **4. Whitelist**
```bash
npx npm-cleaner --keep=dotenv,cors

# Never remove these packages even if unused
# Useful for packages loaded dynamically
```

### **5. Report Only**
```bash
npx npm-cleaner --report

# Creates a report file: npm-cleaner-report.txt
# Share with your team
```

---

## **Output Examples**

### **Clean Project (nothing to remove)**
```bash
$ npx npm-cleaner

🔍 Scanning your project...

✅ Your project is clean!
   All 45 packages in package.json are being used.

No action needed. Great job! 🎉
```

### **Messy Project (lots to clean)**
```bash
$ npx npm-cleaner

🔍 Scanning your project...

⚠️  Found 30 UNUSED packages out of 80 total!

UNUSED PACKAGES (safe to remove):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1.  moment           2.9 MB   ❌ Never used
2.  request          5.1 MB   ❌ Package deprecated
3.  axios            1.2 MB   ❌ Never used
4.  lodash           2.3 MB   ⚠️  Last used: 6 months ago
5.  bluebird         0.8 MB   ❌ Never used
6.  async            0.6 MB   ❌ Never used
7.  validator        0.4 MB   ❌ Never used
8.  node-fetch       0.3 MB   ❌ Never used
9.  chalk            0.5 MB   ❌ Never used
10. debug            0.2 MB   ❌ Never used
...20 more

💾 Total waste: 47.3 MB (37% of node_modules)
⏱️  npm install could be 15 seconds faster
💰 Could save $5.40/month in CDN costs

Remove all unused packages? [y/N]
```

### **With Warnings**
```bash
$ npx npm-cleaner

🔍 Scanning your project...

UNUSED: 10 packages
MAYBE UNUSED: 3 packages

⚠️  WARNING: These packages might be used dynamically:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• dotenv  - Often loaded with require('dotenv/config')
• cors    - Might be used in middleware
• helmet  - Might be used in middleware

Recommendation: Keep these unless you're sure they're unused.

Remove 10 confirmed unused packages? [y/N]
```

---

## **Configuration File (Optional)**

Create `.npm-cleaner.json`:

```json
{
  "exclude": [
    "dotenv",
    "cors",
    "helmet"
  ],
  "scanPaths": [
    "src",
    "lib",
    "scripts"
  ],
  "ignore": [
    "node_modules",
    "dist",
    "build"
  ],
  "autoRemove": false,
  "verbose": false
}
```

---

## **Why This Is Perfect for Beginners**

### **1. Super Simple Problem**
Every developer has unused packages. 100% relatable.

### **2. Easy to Build**
- Read `package.json` ✅ Simple file reading
- Scan files ✅ Loop through files
- Find imports ✅ Basic regex
- Remove packages ✅ Run `npm uninstall`

**Total complexity: 300-500 lines of code**

### **3. Immediate Value**
- Run it → See instant results
- Remove packages → Free disk space
- Feel productive immediately

### **4. No Competition**
Current tools:
- `depcheck` - Complex, hard to use, poor output
- `npm-check` - Outdated, not maintained
- **No tool does this SIMPLY**

### **5. Easy to Test**
```bash
# Create test project
mkdir test-project
cd test-project
npm init -y
npm install moment axios lodash

# Create empty file (don't use packages)
echo "console.log('hello')" > index.js

# Run cleaner
npx npm-cleaner

# Should find all 3 packages as unused ✅
```

---

## **Technical Implementation (Simple)**

### **File Structure**
```
npm-cleaner/
├── bin/
│   └── cli.js          # CLI entry point (50 lines)
├── src/
│   ├── scanner.js      # Scan files for imports (100 lines)
│   ├── analyzer.js     # Compare packages (50 lines)
│   ├── remover.js      # Remove packages (50 lines)
│   └── reporter.js     # Format output (100 lines)
├── package.json
└── README.md
```

### **Core Code (scanner.js)**

```javascript
// Find all JavaScript files
function findJavaScriptFiles(dir) {
  const files = [];
  const items = fs.readdirSync(dir);
  
  items.forEach(item => {
    const path = `${dir}/${item}`;
    
    if (fs.statSync(path).isDirectory()) {
      if (item !== 'node_modules') {
        files.push(...findJavaScriptFiles(path));
      }
    } else if (item.endsWith('.js') || item.endsWith('.ts')) {
      files.push(path);
    }
  });
  
  return files;
}

// Find all imports in a file
function findImports(filePath) {
  const content = fs.readFileSync(filePath, 'utf8');
  const imports = [];
  
  // Find require('package')
  const requireMatches = content.match(/require\(['"]([^'"]+)['"]\)/g) || [];
  requireMatches.forEach(match => {
    const pkg = match.match(/require\(['"]([^'"]+)['"]\)/)[1];
    imports.push(pkg);
  });
  
  // Find import ... from 'package'
  const importMatches = content.match(/import .+ from ['"]([^'"]+)['"]/g) || [];
  importMatches.forEach(match => {
    const pkg = match.match(/from ['"]([^'"]+)['"]/)[1];
    imports.push(pkg);
  });
  
  return imports;
}

// Get package name (remove subpaths)
function getPackageName(importPath) {
  // 'express' -> 'express'
  // 'lodash/get' -> 'lodash'
  // '@babel/core' -> '@babel/core'
  
  if (importPath.startsWith('@')) {
    return importPath.split('/').slice(0, 2).join('/');
  }
  return importPath.split('/')[0];
}
```

### **Core Code (analyzer.js)**

```javascript
function analyzePackages(projectPath) {
  // Read package.json
  const packageJson = JSON.parse(
    fs.readFileSync(`${projectPath}/package.json`, 'utf8')
  );
  
  const installedPackages = [
    ...Object.keys(packageJson.dependencies || {}),
    ...Object.keys(packageJson.devDependencies || {})
  ];
  
  // Scan all files
  const files = findJavaScriptFiles(`${projectPath}/src`);
  const allImports = [];
  
  files.forEach(file => {
    const imports = findImports(file);
    allImports.push(...imports);
  });
  
  // Get unique package names
  const usedPackages = [...new Set(
    allImports.map(getPackageName)
  )];
  
  // Find unused
  const unusedPackages = installedPackages.filter(
    pkg => !usedPackages.includes(pkg)
  );
  
  return {
    total: installedPackages.length,
    used: usedPackages,
    unused: unusedPackages
  };
}
```

### **Core Code (remover.js)**

```javascript
function removePackages(packages) {
  console.log(`Removing ${packages.length} packages...`);
  
  packages.forEach(pkg => {
    console.log(`  ✓ Removing ${pkg}...`);
    
    try {
      execSync(`npm uninstall ${pkg}`, { 
        stdio: 'ignore' 
      });
    } catch (err) {
      console.log(`  ✗ Failed to remove ${pkg}`);
    }
  });
  
  console.log('\n✅ Done!');
}
```

**That's literally it!** The entire tool in ~350 lines of simple code.

---

## **Business Model (Simple)**

### **100% Free & Open Source**

Why?
- Build portfolio
- Learn JavaScript
- Help developers
- Get GitHub stars
- Get hired (companies see your work)

**No need to monetize as a beginner.**

Later (if it gets popular):
- Add Pro features (reports, CI/CD integration)
- Offer consulting
- Or just keep it free for the community ❤️

---

## **Marketing (Simple)**

### **Launch Day**
1. Post on Reddit (r/node, r/javascript)
2. Tweet with before/after screenshot
3. Post on Dev.to with tutorial
4. Submit to Product Hunt

### **Example Tweet**
```
Just cleaned 30 unused packages from my project! 🧹

Before: 500MB node_modules
After: 350MB (-30%)

Check out npm-cleaner - finds unused packages automatically

npx npm-cleaner

[screenshot of output]

#nodejs #javascript #webdev
```

### **Growth Loop**
Every time someone runs it:
1. They see the tool name in output
2. They share their savings ("Freed 50MB!")
3. Their friends try it
4. **Viral growth** 🚀

---

## **Why This Will Succeed**

✅ **Simple problem** - Everyone has it
✅ **Simple solution** - One command
✅ **Immediate value** - See results instantly  
✅ **Easy to build** - 300-500 lines
✅ **No competition** - Existing tools suck
✅ **Viral potential** - People share savings
✅ **Portfolio worthy** - Shows you can ship
✅ **Learn by doing** - File I/O, regex, CLI
✅ **Helpful to community** - Solves real pain

---

## **Getting Started (Your First Steps)**

### **Week 1: Build MVP**
```bash
mkdir npm-cleaner
cd npm-cleaner
npm init -y

# Create basic files
touch bin/cli.js
touch src/scanner.js
touch src/analyzer.js

# Start coding!
```

### **Week 2: Test & Polish**
- Test on your own projects
- Test on open source projects
- Fix bugs
- Make output pretty

### **Week 3: Launch**
- Write README
- Create demo video
- Post everywhere
- Get feedback

**Total time: 3 weeks part-time**

---

## **Example README (For Your GitHub)**

```markdown
# npm-cleaner 🧹

Find and remove unused NPM packages automatically.

## Why?

Your `node_modules` is probably full of packages you installed once and forgot about. This tool finds them and removes them.

## Usage

```bash
npx npm-cleaner
```

That's it!

## Example

```bash
$ npx npm-cleaner

Found 60 packages
Used: 35 ✅
Unused: 25 ⚠️

Remove 25 unused packages? [y/N] y

✅ Freed 47MB of disk space!
```

## Install

```bash
npm install -g npm-cleaner
```

## Features

- ✅ Finds unused packages
- ✅ Safe by default (asks before removing)
- ✅ Shows disk space saved
- ✅ Fast (scans in seconds)
- ✅ Zero configuration

## License

MIT
```

---

**This is PERFECT for a beginner:**
- Simple problem ✅
- Simple code ✅
- Real value ✅
- Portfolio-worthy ✅
- Fun to build ✅

**Want to build this one? I can help you write the code step by step!**
