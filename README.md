# n8n-experiments

---

## Process_rows_in_CSV_file

<img width="818" height="276" alt="image" src="https://github.com/user-attachments/assets/7fb61422-5449-48b5-bdc9-9eaf361581af" />

This example starts with the user uploading a .csv file with the following columns
- Produce Item
- In Stock Count
- Last Arrival Date

n8n cannot process .csv data natively, so a Code Node to parse the rows into n8n items is needed (set mode to 'Run Once for **All** Items'):
```
const binaryKey = 'file';

const buffer = await this.helpers.getBinaryDataBuffer(0, binaryKey);
let text = buffer.toString('utf8').replace(/^\uFEFF/, '');

const lines = text.split(/\r?\n/).filter(line => line.trim() !== '');

function parseCsvLine(line) {
  const result = [];
  let current = '';
  let inQuotes = false;

  for (let i = 0; i < line.length; i++) {
    const char = line[i];
    const next = line[i + 1];

    if (char === '"' && inQuotes && next === '"') {
      current += '"';
      i++;
    } else if (char === '"') {
      inQuotes = !inQuotes;
    } else if (char === ',' && !inQuotes) {
      result.push(current);
      current = '';
    } else {
      current += char;
    }
  }

  result.push(current);
  return result;
}

function convertValue(value) {
  const trimmed = value.trim();

  if (trimmed === '') {
    return '';
  }

  // Convert integers and decimals, including negative numbers
  if (/^-?\d+(\.\d+)?$/.test(trimmed)) {
    return Number(trimmed);
  }

  return trimmed;
}

if (lines.length < 2) {
  return [];
}

const headers = parseCsvLine(lines[0]).map(h => h.trim());

return lines.slice(1).map(line => {
  const values = parseCsvLine(line);
  const row = {};

  headers.forEach((header, i) => {
    row[header] = convertValue(values[i] ?? '');
  });

  return {
    json: row
  };
});
```
