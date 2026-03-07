# n8n-experiments

---

## Process_rows_in_CSV_file ([template link](https://github.com/developer-nome/n8n-experiments/blob/main/workflow_templates/Process_rows_in_CSV_file.json))

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
The AI Agent node can use values from the previous node in the Prompt field with an expression:
<img width="692" height="371" alt="image" src="https://github.com/user-attachments/assets/3773c208-a7c4-4ed7-b4f8-71c45e99ca50" />
The Merge node then utilizes the data from the node before the AI Agent to perserve the values for saving a new .csv file later
<img width="617" height="398" alt="image" src="https://github.com/user-attachments/assets/7e6347e3-b162-439d-9a1a-175291829635" />
The following Code Node is for combinig the original values with the output of the AI Agent into 1 json object (mode = Run Once for Each Item):
```
return {
  json: {
    produce_item: $json["Produce Item"],
    in_stock_count: $json["In Stock Count"],
    last_arrival_date: $json["Last Arrival Date"],
    haiku: $json["output"]
  }
};
```

---
---

## Talk with your DataTable using LMStudio ([template link](https://github.com/developer-nome/n8n-experiments/blob/main/workflow_templates/Talk%20with%20your%20DataTable%20using%20LMStudio.json))

<img width="463" height="255" alt="image" src="https://github.com/user-attachments/assets/fd5b7822-c2b7-4ab3-84d9-5d40adc45c6c" />

This example shows how to use model provider that is not listed in n8n's list (must be compatible with OpenAI's API structure) and that you can query n8n Data Tables. The AI Agent nodes's System Message should mention that the agent has the ability to tool call the function, *insertFunctionNameHere* that uses the n8n data talbe called *useTableNameHere*.


