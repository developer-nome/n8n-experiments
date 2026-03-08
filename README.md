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

This example shows how to use model provider that is not listed in n8n's list (as long as it is compatible with OpenAI's API structure) and that you can query n8n Data Tables. The **AI Agent nodes's** System Message should mention that the agent has the ability to tool call the function, *insertFunctionNameHere* that uses the n8n data talbe called *useTableNameHere*.

In the **OpenAI Chat Model node's** parameters under 'Credential to connect with', click the pencil icon and set the appropriate base url:

<img width="587" height="540" alt="image" src="https://github.com/user-attachments/assets/0c7fb201-f227-40b9-a7c2-7d9e2cb609b0" />

You will also need to *disable* 'Use Responses API'

<img width="318" height="274" alt="image" src="https://github.com/user-attachments/assets/215600aa-9769-41eb-8248-0047bae9def1" />

In the **getRowsInDataTable node** you will need to select which table to use:

<img width="324" height="545" alt="image" src="https://github.com/user-attachments/assets/75282a9f-a48f-486d-ad7c-96f3b5f9447c" />

---
---

## TellAgentToInsertDataUsingOllama ([template link](https://github.com/developer-nome/n8n-experiments/blob/main/workflow_templates/TellAgentToInsertDataUsingOllama.json))

<img width="703" height="443" alt="image" src="https://github.com/user-attachments/assets/542d7673-1c57-4004-a52b-d7bff945b533" />

This example shows how to use a human-in-the-loop (HITL) to selectivly insert rows into an n8n Data Table. For the **Chat node** following the AI Agent output to work, you must set the 1st chat trigger node's parameter option for 'Response Mode' to 'Using Response Nodes':

<img width="390" height="296" alt="image" src="https://github.com/user-attachments/assets/a0492eab-2a42-4a60-a6a6-289f2e5688b7" />

In the **AI Agent node**, you need to set the 'System Message' in a way that tells the agent which tools it has access to and the table schema in json format:

<img width="357" height="429" alt="image" src="https://github.com/user-attachments/assets/aa10bc7d-49ae-4178-9cd8-d28e675cdd87" />

For the 2nd **Chat node** to be used by the human to approve or deny the insertion, you will need to set the 'Message' to 'The agent wants to call {{ $tool.name }}' and use the 'Approval Options' to give the human workflow options:

<img width="354" height="507" alt="image" src="https://github.com/user-attachments/assets/6e78b6ec-1242-4694-a94a-b311b17f5981" />

In the parameters for the **insertRowInDataTable node**, you will need to configure the items as shown in the screenshot below:

<img width="342" height="633" alt="image" src="https://github.com/user-attachments/assets/000730e3-5a14-4803-8f18-abb410761180" />


