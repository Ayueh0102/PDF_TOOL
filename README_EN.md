# 📄 PDF Toolset

A pure frontend PDF processing toolset. All operations are performed in your browser, with no server uploads required, ensuring your privacy and data security.

**Author:** CHUN TING YUEH

**Languages:** [繁體中文](README.md) | [English](README_EN.md)

---

## ✨ Features

### 🖼️ Image to PDF Converter
Convert multiple images into a single PDF file.

**Key Features:**
- Support for multiple image formats (JPG, PNG, GIF, WebP, etc.)
- Drag-and-drop upload for easy operation
- Customize image order
- Adjust image quality and page size
- Preview function

**How to Use:**
1. Click or drag images to the upload area
2. Adjust image order (draggable)
3. Set page size and image quality
4. Click the "Generate PDF" button
5. Download the generated PDF file

---

### 🔄 PDF/Word to PNG Converter
Convert PDF or Word documents to high-quality PNG images.

**Key Features:**
- Support for PDF and Word documents (.doc, .docx)
- Custom resolution (72-300 DPI)
- Batch download all pages
- Single page download option

**How to Use:**
1. Upload a PDF or Word document
2. Set output resolution
3. Wait for conversion to complete
4. Download individual pages or all images

---

### 🔓 PDF Unlock Tool
Remove password protection from PDF files and output high-quality unlocked PDFs.

**Key Features:**
- Remove password protection from PDFs
- Automatic 300 DPI high-quality output
- Lossless PNG compression
- Maintain original page dimensions

**How to Use:**
1. Upload a password-protected PDF file
2. Enter the correct password
3. Click the "Unlock PDF" button
4. Wait for processing to complete
5. Download the unlocked PDF file

**Important Notes:**
- The process converts PDF to high-quality images and regenerates them to ensure the best visual quality
- Processing time depends on page count and complexity

---

### ✂️ PDF Split Tool
Split PDF files into individual pages or custom groups.

**Key Features:**
- **Four Split Modes:**
  - 📄 **Single Page**: Export each page as an individual PDF
  - 📚 **Group by N Pages**: Combine every N pages into one PDF (e.g., every 5 pages)
  - 📖 **Page Range**: Select a specific page range
  - 🎯 **Custom Pages**: Specify any page combination (e.g., 1,3,5,7-10)
- **Two-Stage Processing:**
  - Stage 1: Quick low-DPI preview generation (72 DPI)
  - Stage 2: On-demand high-DPI output generation (300 DPI)
- Custom file naming format
- Batch download or individual download

**How to Use:**
1. Upload a PDF file
2. Wait for preview generation
3. Select split mode and configure parameters:
   - **Single Page**: No configuration needed, generate directly
   - **Group by N Pages**: Enter pages per group (e.g., 5)
   - **Page Range**: Enter start and end pages
   - **Custom Pages**: Enter page numbers (e.g., 1,3,5,7-10)
4. Select file naming format
5. Click the "🚀 Generate PDF" button
6. Wait for high-quality generation to complete
7. Download files

**Examples:**
- Group by 5 pages: A 30-page PDF will generate 6 files (pages_1-5.pdf, pages_6-10.pdf...)
- Page range: Select pages 10-20, can choose to merge or separate output
- Custom pages: Enter "1,5,10-15" to extract only these pages

---

## 🚀 Technical Features

### Pure Frontend Implementation
- ✅ 100% client-side processing
- ✅ No server required
- ✅ Privacy protection, files not uploaded
- ✅ Completely free to use

### Technologies Used
- **jsPDF (2.5.1)** - PDF generation
- **PDF.js (3.11.174)** - PDF parsing and rendering
- **Mammoth.js (1.4.2)** - Word document processing
- **html2canvas (1.4.1)** - HTML to image conversion

### Quality Assurance
- 🎨 300 DPI high-quality output
- 🖼️ Lossless PNG compression
- 📐 Maintain original aspect ratio
- ⚡ Optimized rendering engine

---

## 📦 How to Use

### Online Use
Simply open the corresponding HTML file:
- `index.html` - Main page (all tools navigation)
- `image_to_pdf_converter.html` - Image to PDF
- `pdf_to_png_converter.html` - PDF/Word to PNG
- `pdf_unlock_converter.html` - PDF Unlock
- `pdf_split_converter.html` - PDF Split

### Local Deployment
1. Clone or download this project
2. Open any HTML file in your browser
3. No installation of dependencies or server execution required

### Browser Requirements
- ✅ Chrome (Recommended)
- ✅ Firefox
- ✅ Safari
- ✅ Edge

Requires a browser that supports modern JavaScript (ES6+).

---

## 🎨 Interface Design

- Modern gradient color design
- Responsive layout, mobile device support
- Intuitive drag-and-drop operation
- Real-time progress feedback
- Friendly error messages

---

## 📝 Important Notes

1. **File Size Limit**: Due to browser memory limitations, it's recommended not to process PDFs with more than 100 pages or files larger than 50MB at once
2. **Processing Time**: Depends on file size, page count, and your device performance
3. **Privacy & Security**: All processing is done in your browser, files are not uploaded to any server
4. **Quality Settings**: Using 300 DPI provides the best print quality but increases file size

---

## 🛠️ Development

### Project Structure
```
PDF_TOOL/
├── index.html                      # Main page
├── image_to_pdf_converter.html     # Image to PDF
├── pdf_to_png_converter.html       # PDF/Word to PNG
├── pdf_unlock_converter.html       # PDF Unlock
├── pdf_split_converter.html        # PDF Split
├── README.md                        # Documentation (Chinese)
└── README_EN.md                     # Documentation (English)
```

### Development Principles
- Maintain pure frontend implementation
- Maintain modern UI design
- Ensure good user experience
- All user-facing text in Traditional Chinese (for Chinese version)

---

## 📄 License

This project is for learning and personal use only.

---

## 🤝 Contribution

Questions and suggestions are welcome!

---

## 👤 Author

**CHUN TING YUEH**

---

## 📮 Contact

Feel free to reach out with any questions or suggestions.

---

**Enjoy a convenient PDF processing experience!** 🎉

## 🔗 Third-Party Licenses

See `THIRD_PARTY_LICENSES.md` for third-party open-source packages and their licenses used in this project. PDF.js is licensed under Apache-2.0 and requires preserving its LICENSE and NOTICE.

- Project License: `LICENSE`
- Third-Party Licenses Summary: `THIRD_PARTY_LICENSES.md`

---

