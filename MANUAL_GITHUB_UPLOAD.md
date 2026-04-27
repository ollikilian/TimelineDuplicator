# Manual GitHub Upload Guide

This guide shows how to manually upload all files to GitHub using the web interface (no Git CLI needed).

## Step 1: Create Empty Repository on GitHub

1. Go to [github.com/new](https://github.com/new)
2. **Repository name**: `TimelineDuplicator`
3. **Description**: "A Pixera Control module for duplicating timelines and replacing clip resources"
4. **Visibility**: Select **Public** (recommended for community modules) or **Private**
5. **Initialize this repository with**:
   - ❌ Do NOT check "Add a README file"
   - ❌ Do NOT check "Add .gitignore"
   - ❌ Do NOT check "Choose a license"
6. Click **Create repository**

You'll see an empty repo with a message like:
```
...or create a new repository on the command line
```

**Copy your repo URL**, e.g.: `https://github.com/YOUR-USERNAME/TimelineDuplicator`

---

## Step 2: Upload Files via GitHub Web Interface

### 2a. Create the Root-Level Files

1. Go back to your repo homepage: `https://github.com/YOUR-USERNAME/TimelineDuplicator`
2. Click **Add file** → **Create new file**

**Create: `README.md`**
- Copy the content from **README.md** (included in download)
- Paste into the text field
- Scroll down, enter commit message: `Add: README documentation`
- Click **Commit new file**

**Create: `CHANGELOG.md`**
- Repeat process with **CHANGELOG.md**
- Commit message: `Add: CHANGELOG`

**Create: `CONTRIBUTING.md`**
- Repeat process with **CONTRIBUTING.md**
- Commit message: `Add: CONTRIBUTING guidelines`

**Create: `LICENSE`**
- Repeat process with **LICENSE**
- Commit message: `Add: MIT License`

**Create: `.gitignore`**
- Repeat process with **.gitignore**
- Commit message: `Add: .gitignore`

---

### 2b. Upload the Module File

1. Click **Add file** → **Upload files**
2. Drag & drop **TimelineDuplicator.json** into the upload area
3. Commit message: `Add: TimelineDuplicator module v1.0`
4. Click **Commit changes**

---

### 2c. Create Docs Folder & Upload Documentation

1. Click **Add file** → **Create new file**
2. In the filename field, enter: `docs/README.md`
   - This creates the `docs/` folder automatically
3. Enter a placeholder message: `# Documentation`
4. Commit: `Create docs folder`

5. Now click **Add file** → **Upload files**
6. Navigate to the `docs/` folder (click "docs" in breadcrumb)
7. Drag & drop **TimelineDuplicator_Documentation.docx**
8. Commit message: `Add: Complete user documentation (Word format)`
9. Click **Commit changes**

---

## Step 3: Verify Your Repository

Go to your repo homepage and verify:

✅ README.md is visible  
✅ TimelineDuplicator.json is listed  
✅ docs/ folder exists  
✅ TimelineDuplicator_Documentation.docx is in docs/  
✅ LICENSE file is present  
✅ CHANGELOG.md visible  
✅ CONTRIBUTING.md visible  
✅ .gitignore present  

---

## Step 4: (Optional) Create a Release

Once files are uploaded:

1. Go to **Releases** tab (right sidebar)
2. Click **Create a new release**
3. **Tag version**: `v1.0.0`
4. **Release title**: `TimelineDuplicator v1.0`
5. **Description**:
   ```markdown
   ## TimelineDuplicator v1.0

   Initial release of the TimelineDuplicator module for Pixera Control.

   ### Features
   - Single timeline duplication with resource replacement
   - Batch processing for multi-language workflows
   - Two match modes: By Order and By Name
   - Dynamic dropdown selection
   - Comprehensive error handling and logging

   ### Installation
   Download `TimelineDuplicator.json` and copy to:
   ```
   C:\ProgramData\AV Stumpfl\Pixera\control_library_user\
   ```

   ### Documentation
   See `docs/TimelineDuplicator_Documentation.docx` for complete user guide.

   ### Requirements
   - Pixera 2.0.172+
   - API Revision 481+
   ```
6. **Attach files** (optional):
   - Click "Attach binaries by dropping them here..."
   - Drag & drop:
     - TimelineDuplicator.json
     - TimelineDuplicator_Documentation.docx
7. Click **Publish release**

---

## Step 5: Share Your Repository

1. Copy your repo URL: `https://github.com/YOUR-USERNAME/TimelineDuplicator`
2. Share in:
   - Pixera community forums
   - AV Stumpfl support channels
   - Social media / professional networks
3. (Optional) Add GitHub **Topics**:
   - Click **About** (right sidebar)
   - Add tags: `pixera`, `pixera-control`, `automation`, `media-server`

---

## File Organization Reference

When you're done, your repository should look like:

```
TimelineDuplicator/
│
├─ README.md                          ← Create/Upload
├─ CHANGELOG.md                       ← Create/Upload
├─ CONTRIBUTING.md                    ← Create/Upload
├─ LICENSE                            ← Create/Upload
├─ .gitignore                         ← Create/Upload
├─ TimelineDuplicator.json            ← Upload
│
└─ docs/
   └─ TimelineDuplicator_Documentation.docx  ← Upload
```

---

## Troubleshooting

### "File already exists"
- You're trying to create a file that's already there
- Skip and move to the next file

### Can't find Upload button
- Click **Add file** dropdown (green button)
- Select **Upload files**

### Docs folder not created
- When creating a file in GitHub, use path: `docs/filename`
- This auto-creates the folder

### File didn't upload
- Check file size (should be < 25 MB)
- Try uploading again
- Refresh the page if stuck

---

## Summary

You now have a professional, complete GitHub repository for TimelineDuplicator:

✅ Pixera Control module (JSON)  
✅ User documentation (Word + Markdown)  
✅ Open-source license (MIT)  
✅ Contribution guidelines  
✅ Version history (CHANGELOG)  
✅ Professional README with examples  

**Next steps:**
1. Share the repo link
2. Monitor Issues & Discussions for feedback
3. Create releases for future versions

---

**Questions?** Refer to:
- 📖 [GitHub Help](https://docs.github.com)
- 🎓 [Pixera Help Center](https://help.pixera.one)

**Done!** Your TimelineDuplicator is now public. 🚀
