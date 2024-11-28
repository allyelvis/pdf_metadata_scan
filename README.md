Here’s a Bash script that scans all PDF documents in a specified directory (and optionally subdirectories) to detect the editor and original source using **`pdfinfo`** from the **poppler-utils** package.

### Script: `scan_pdf_metadata.sh`

```bash
#!/bin/bash

# Check if pdfinfo is installed
if ! command -v pdfinfo &>/dev/null; then
    echo "Error: 'pdfinfo' is not installed. Install it using 'sudo apt-get install poppler-utils' or equivalent."
    exit 1
fi

# Directory to scan
DIR_TO_SCAN=${1:-.} # Default to current directory if no argument is passed

# Output file for results
OUTPUT_FILE="pdf_metadata_scan_results.txt"
> "$OUTPUT_FILE" # Clear or create the file

# Function to scan a single PDF
scan_pdf() {
    local pdf_file="$1"

    # Extract metadata
    echo "Scanning: $pdf_file"
    local metadata=$(pdfinfo "$pdf_file" 2>/dev/null)

    if [[ -z "$metadata" ]]; then
        echo "Error: Unable to read metadata for $pdf_file" >> "$OUTPUT_FILE"
        return
    fi

    # Parse important fields
    local title=$(echo "$metadata" | grep -i "^Title:" | cut -d ':' -f 2- | xargs)
    local author=$(echo "$metadata" | grep -i "^Author:" | cut -d ':' -f 2- | xargs)
    local producer=$(echo "$metadata" | grep -i "^Producer:" | cut -d ':' -f 2- | xargs)
    local creator=$(echo "$metadata" | grep -i "^Creator:" | cut -d ':' -f 2- | xargs)
    local creation_date=$(echo "$metadata" | grep -i "^CreationDate:" | cut -d ':' -f 2- | xargs)

    # Write results to output file
    {
        echo "File: $pdf_file"
        echo "  Title: $title"
        echo "  Author: $author"
        echo "  Producer (Editor): $producer"
        echo "  Creator (Original Source): $creator"
        echo "  Creation Date: $creation_date"
        echo ""
    } >> "$OUTPUT_FILE"
}

# Find and process all PDF files
echo "Scanning PDF files in directory: $DIR_TO_SCAN"
find "$DIR_TO_SCAN" -type f -name "*.pdf" | while read -r pdf; do
    scan_pdf "$pdf"
done

echo "Scan completed. Results saved to $OUTPUT_FILE."
```

---

### How to Use
1. Save the script as `scan_pdf_metadata.sh`.
2. Make it executable:
   ```bash
   chmod +x scan_pdf_metadata.sh
   ```
3. Run the script with the directory you want to scan as an argument:
   ```bash
   ./scan_pdf_metadata.sh /path/to/directory
   ```
   If no directory is specified, the script will default to the current directory.

---

### Output Example
For each PDF file, the script will extract the following information:
- **Title**: Document title, if available.
- **Author**: Document author, if available.
- **Producer (Editor)**: The tool or software used to edit the document.
- **Creator (Original Source)**: The original tool or application that created the document.
- **Creation Date**: Date the document was created.

Results are saved in `pdf_metadata_scan_results.txt`.
