# Natural Language to SQL Query Matcher

A machine learning system that converts natural language queries to SQL using BERT embeddings and FAISS similarity search. Works with your existing CSV datasets to automatically generate SQL query templates and match them with natural language requests.

## Features

- **Automatic Schema Detection**: Analyzes your CSV files to understand column types and generate appropriate SQL templates
- **Template-based SQL Query Generation**: Creates hundreds of SQL query variations from your data structure
- **Natural Language Processing**: Uses BERT embeddings to understand and match natural language queries
- **Fast Similarity Search**: Employs FAISS for efficient vector similarity matching
- **Multi-dataset Support**: Works with any CSV dataset - automatically adapts to your schema
- **Performance Benchmarking**: Includes timing analysis for embedding and indexing operations
- **Dataset Analysis Tools**: Comprehensive analysis and validation of your CSV files

## Quick Start

### 1. Setup
```bash
git clone https://github.com/yourusername/nl-to-sql-matcher.git
cd nl-to-sql-matcher
pip install -r requirements.txt
```

### 2. Add Your Data
Place your CSV files in the `data/` directory:
```
data/
├── inventory.csv
├── students.csv
├── bugReport.csv
└── your_custom_dataset.csv
```

### 3. Analyze Your Data
```bash
python src/dataset_analyzer.py
```
This will analyze all your CSV files and generate a report showing:
- Dataset structure and column types
- Data quality issues
- Generated sample queries
- Compatibility with the system

### 4. Run the System
```python
from src.matcher import QueryMatcher
import pandas as pd

# Load your data
df = pd.read_csv('data/your_dataset.csv')

# Initialize the matcher (automatically detects schema)
matcher = QueryMatcher(df, dataset_name='your_dataset.csv')

# Query in natural language
matches = matcher.find_matches("Show me records where price is greater than 100", top_k=5)

for match, similarity in matches:
    print(f"SQL: {match[1]} (Similarity: {similarity:.4f})")
```

### 5. Test All Your Datasets
```bash
python examples/usage_example.py
```

## How It Works

### 1. **Schema Detection**
The system automatically analyzes your CSV files to:
- Detect column data types (numeric vs text)
- Identify unique values in text columns
- Calculate ranges for numeric columns
- Generate appropriate SQL templates

### 2. **Template Generation**
Based on your schema, the system creates SQL templates like:
```sql
SELECT * FROM your_table WHERE numeric_column > [value]
SELECT text_column FROM your_table WHERE other_column = [value]
SELECT * FROM your_table WHERE col1 > [val1] AND col2 = [val2]
```

### 3. **Natural Language Mapping**
Each SQL template is converted into natural language variants:
```
"Show me records where price is greater than 50"
"Give me items with price more than 50"
"I want products where price > 50"
```

### 4. **Embedding & Search**
- BERT creates vector representations of all natural language variants
- FAISS indexes these vectors for fast similarity search
- User queries are matched against the most similar variants

## Supported Data Types

The system automatically handles:
- **Numeric columns** (int64, float64): Generates comparison queries (>, <, =, >=, <=, !=)
- **Text columns** (object): Generates equality and inequality queries
- **Mixed queries**: Combines multiple columns with AND/OR logic

## Dataset Examples

### Inventory Management
```csv
productID,name,price,stock,category
P001,Widget A,15.99,25,Electronics
P002,Widget B,8.50,12,Home
```
Natural language: *"Show me electronics with price greater than 10"*

### Student Records
```csv
studentId,name,gpa,division,age
S001,John Smith,3.75,A,20
S002,Jane Doe,3.92,B,19
```
Natural language: *"Give me students from division A with GPA above 3.5"*

### Bug Tracking
```csv
bugId,severity,assignedTo,status,reportedBy
BUG001,High,Sarah,Open,John
BUG002,Medium,Mike,Closed,Alice
```
Natural language: *"Show me high severity bugs assigned to Sarah"*

## Performance

Typical performance on a modern CPU:
- **1000 queries**: ~2-3 seconds embedding + indexing
- **Search time**: ~5-10ms per query
- **Index size**: ~10-50MB depending on dataset size
- **GPU acceleration**: Automatic CUDA detection for faster embedding

## Project Structure
```
nl-to-sql-matcher/
├── data/                    # Your CSV files go here
├── src/
│   ├── matcher.py          # Main query matching system
│   ├── query_generator.py  # SQL template generation
│   ├── embeddings.py       # BERT embedding utilities
│   ├── data_loader.py      # CSV loading and validation
│   └── dataset_analyzer.py # Dataset analysis tools
├── config/
│   └── templates.py        # SQL templates and mappings
├── examples/
│   └── usage_example.py    # Usage demonstrations
└── models/                 # Saved models and indexes
```

## Advanced Usage

### Custom Token Mappings
Add custom natural language mappings for your domain:
```python
custom_tokens = {
    "price": ["cost", "amount", "fee", "charge"],
    "quantity": ["stock", "inventory", "amount", "count"],
    "SELECT": ["show me", "give me", "find", "get"]
}
```

### Batch Processing
Process multiple datasets:
```python
from src.data_loader import DataLoader

loader = DataLoader("data/")
for dataset in loader.discover_datasets():
    df = loader.load_dataset(dataset)
    matcher = QueryMatcher(df, dataset)
    # Process each dataset...
```

### Performance Optimization
```python
# Use GPU if available
matcher = QueryMatcher(df, dataset_name, model_name="bert-base-uncased")

# Save/load trained models
matcher.save_index("models/my_dataset")
matcher.load_index("models/my_dataset")
```

## Requirements

- Python 3.7+
- PyTorch
- Transformers (Hugging Face)
- FAISS-CPU (or FAISS-GPU for CUDA acceleration)
- Pandas, NumPy

## Contributing

1. Fork the repository
2. Add your CSV files to test with different schemas
3. Create a feature branch (`git checkout -b feature/amazing-feature`)
4. Test with your datasets using `python examples/usage_example.py`
5. Commit your changes (`git commit -m 'Add support for new data type'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

