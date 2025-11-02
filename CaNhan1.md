<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <link rel="icon" href="https://public-frontend-cos.metadl.com/mgx/img/favicon.png" type="image/png">
  <title>Code Generation & Review App</title>
  <meta name="description" content="Advanced Code Generation and Review Platform" />
  <meta name="author" content="MGX" />

  <meta property="og:title" content="Code Generation & Review App" />
  <meta property="og:description" content="Advanced Code Generation and Review Platform" />
  <meta property="og:type" content="website" />

</head>

<body>
  <div id="root"></div>
  <script type="module" src="/src/main.tsx"></script>
</body>







</html>
export interface CodeTemplate {
  id: string;
  name: string;
  language: string;
  description: string;
  code: string;
  category: string;
}

export const codeTemplates: CodeTemplate[] = [
  {
    id: 'react-component',
    name: 'React Functional Component',
    language: 'typescript',
    description: 'Basic React functional component with TypeScript',
    category: 'React',
    code: `import React from 'react';

interface Props {
  title: string;
  children?: React.ReactNode;
}

const MyComponent: React.FC<Props> = ({ title, children }) => {
  return (
    <div className="p-4">
      <h2 className="text-xl font-bold">{title}</h2>
      {children}
    </div>
  );
};

export default MyComponent;`
  },
  {
    id: 'express-api',
    name: 'Express API Route',
    language: 'javascript',
    description: 'Basic Express.js API route handler',
    category: 'Node.js',
    code: `const express = require('express');
const router = express.Router();

// GET route
router.get('/api/users', async (req, res) => {
  try {
    // Your logic here
    const users = await getUsersFromDB();
    res.json({ success: true, data: users });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// POST route
router.post('/api/users', async (req, res) => {
  try {
    const { name, email } = req.body;
    const newUser = await createUser({ name, email });
    res.status(201).json({ success: true, data: newUser });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
});

module.exports = router;`
  },
  {
    id: 'python-class',
    name: 'Python Class',
    language: 'python',
    description: 'Basic Python class with methods',
    category: 'Python',
    code: `class DataProcessor:
    def __init__(self, data=None):
        self.data = data or []
        self.processed = False
    
    def add_item(self, item):
        """Add an item to the data list"""
        self.data.append(item)
        self.processed = False
    
    def process_data(self):
        """Process the data"""
        if not self.data:
            return []
        
        # Your processing logic here
        processed_data = [item.upper() if isinstance(item, str) else item 
                         for item in self.data]
        self.processed = True
        return processed_data
    
    def get_stats(self):
        """Get statistics about the data"""
        return {
            'total_items': len(self.data),
            'processed': self.processed,
            'types': list(set(type(item).__name__ for item in self.data))
        }`
  },
  {
    id: 'sql-crud',
    name: 'SQL CRUD Operations',
    language: 'sql',
    description: 'Basic CRUD operations in SQL',
    category: 'Database',
    code: `-- Create table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert data
INSERT INTO users (name, email) 
VALUES ('John Doe', 'john@example.com');

-- Select data
SELECT * FROM users WHERE email = 'john@example.com';

-- Update data
UPDATE users 
SET name = 'Jane Doe' 
WHERE id = 1;

-- Delete data
DELETE FROM users WHERE id = 1;

-- Create index for better performance
CREATE INDEX idx_users_email ON users(email);`
  }
];

export const getTemplatesByCategory = (category: string) => {
  return codeTemplates.filter(template => template.category === category);
};

export const getAllCategories = () => {
  return [...new Set(codeTemplates.map(template => template.category))];
};








export interface ReviewRule {
  id: string;
  name: string;
  description: string;
  severity: 'error' | 'warning' | 'info';
  pattern?: RegExp;
  check: (code: string) => ReviewResult[];
}

export interface ReviewResult {
  line: number;
  column: number;
  message: string;
  severity: 'error' | 'warning' | 'info';
  rule: string;
}

export const reviewRules: ReviewRule[] = [
  {
    id: 'no-console-log',
    name: 'No Console Logs',
    description: 'Avoid console.log statements in production code',
    severity: 'warning',
    pattern: /console\.log/g,
    check: (code: string): ReviewResult[] => {
      const results: ReviewResult[] = [];
      const lines = code.split('\n');
      
      lines.forEach((line, index) => {
        if (line.includes('console.log')) {
          results.push({
            line: index + 1,
            column: line.indexOf('console.log') + 1,
            message: 'Remove console.log statement before production',
            severity: 'warning',
            rule: 'no-console-log'
          });
        }
      });
      
      return results;
    }
  },
  {
    id: 'missing-semicolon',
    name: 'Missing Semicolons',
    description: 'JavaScript/TypeScript statements should end with semicolons',
    severity: 'error',
    check: (code: string): ReviewResult[] => {
      const results: ReviewResult[] = [];
      const lines = code.split('\n');
      
      lines.forEach((line, index) => {
        const trimmed = line.trim();
        if (trimmed && 
            !trimmed.endsWith(';') && 
            !trimmed.endsWith('{') && 
            !trimmed.endsWith('}') &&
            !trimmed.startsWith('//') &&
            !trimmed.startsWith('*') &&
            !trimmed.includes('import ') &&
            !trimmed.includes('export ') &&
            trimmed.length > 0) {
          results.push({
            line: index + 1,
            column: trimmed.length,
            message: 'Missing semicolon at end of statement',
            severity: 'error',
            rule: 'missing-semicolon'
          });
        }
      });
      
      return results;
    }
  },
  {
    id: 'long-function',
    name: 'Long Function',
    description: 'Functions should not be too long (max 50 lines)',
    severity: 'warning',
    check: (code: string): ReviewResult[] => {
      const results: ReviewResult[] = [];
      const lines = code.split('\n');
      let functionStart = -1;
      let braceCount = 0;
      
      lines.forEach((line, index) => {
        const trimmed = line.trim();
        
        if (trimmed.includes('function ') || trimmed.includes('=>') || trimmed.match(/^\s*\w+\s*\(/)) {
          functionStart = index;
          braceCount = 0;
        }
        
        if (functionStart !== -1) {
          braceCount += (line.match(/{/g) || []).length;
          braceCount -= (line.match(/}/g) || []).length;
          
          if (braceCount === 0 && functionStart !== -1) {
            const functionLength = index - functionStart + 1;
            if (functionLength > 50) {
              results.push({
                line: functionStart + 1,
                column: 1,
                message: `Function is too long (${functionLength} lines). Consider breaking it down.`,
                severity: 'warning',
                rule: 'long-function'
              });
            }
            functionStart = -1;
          }
        }
      });
      
      return results;
    }
  },
  {
    id: 'no-var',
    name: 'No var keyword',
    description: 'Use let or const instead of var',
    severity: 'warning',
    check: (code: string): ReviewResult[] => {
      const results: ReviewResult[] = [];
      const lines = code.split('\n');
      
      lines.forEach((line, index) => {
        if (line.match(/\bvar\s+/)) {
          results.push({
            line: index + 1,
            column: line.indexOf('var') + 1,
            message: 'Use let or const instead of var',
            severity: 'warning',
            rule: 'no-var'
          });
        }
      });
      
      return results;
    }
  }
];

export const runCodeReview = (code: string): ReviewResult[] => {
  const allResults: ReviewResult[] = [];
  
  reviewRules.forEach(rule => {
    const results = rule.check(code);
    allResults.push(...results);
  });
  
  return allResults.sort((a, b) => a.line - b.line);
};

export const getReviewStats = (results: ReviewResult[]) => {
  const stats = {
    total: results.length,
    errors: results.filter(r => r.severity === 'error').length,
    warnings: results.filter(r => r.severity === 'warning').length,
    info: results.filter(r => r.severity === 'info').length
  };
  
  return stats;
};






import React, { useState, useEffect } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Textarea } from '@/components/ui/textarea';
import { Badge } from '@/components/ui/badge';
import { Copy, Download, Upload } from 'lucide-react';
import { toast } from 'sonner';

interface CodeEditorProps {
  initialCode?: string;
  language?: string;
  onCodeChange?: (code: string) => void;
  readOnly?: boolean;
}

const CodeEditor: React.FC<CodeEditorProps> = ({
  initialCode = '',
  language = 'javascript',
  onCodeChange,
  readOnly = false
}) => {
  const [code, setCode] = useState(initialCode);
  const [lineNumbers, setLineNumbers] = useState<number[]>([]);

  useEffect(() => {
    const lines = code.split('\n').length;
    setLineNumbers(Array.from({ length: lines }, (_, i) => i + 1));
  }, [code]);

  useEffect(() => {
    setCode(initialCode);
  }, [initialCode]);

  const handleCodeChange = (value: string) => {
    setCode(value);
    onCodeChange?.(value);
  };

  const copyToClipboard = async () => {
    try {
      await navigator.clipboard.writeText(code);
      toast.success('Code copied to clipboard!');
    } catch (err) {
      toast.error('Failed to copy code');
    }
  };

  const downloadCode = () => {
    const element = document.createElement('a');
    const file = new Blob([code], { type: 'text/plain' });
    element.href = URL.createObjectURL(file);
    element.download = `code.${language}`;
    document.body.appendChild(element);
    element.click();
    document.body.removeChild(element);
    toast.success('Code downloaded!');
  };

  const uploadFile = () => {
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = '.js,.ts,.jsx,.tsx,.py,.sql,.html,.css,.json';
    input.onchange = (e) => {
      const file = (e.target as HTMLInputElement).files?.[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const content = e.target?.result as string;
          handleCodeChange(content);
          toast.success('File uploaded successfully!');
        };
        reader.readAsText(file);
      }
    };
    input.click();
  };

  return (
    <Card className="w-full">
      <CardHeader className="pb-3">
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-2">
            <CardTitle className="text-lg">Code Editor</CardTitle>
            <Badge variant="outline">{language}</Badge>
          </div>
          <div className="flex gap-2">
            <Button
              variant="outline"
              size="sm"
              onClick={uploadFile}
              className="flex items-center gap-1"
            >
              <Upload className="w-4 h-4" />
              Upload
            </Button>
            <Button
              variant="outline"
              size="sm"
              onClick={copyToClipboard}
              className="flex items-center gap-1"
            >
              <Copy className="w-4 h-4" />
              Copy
            </Button>
            <Button
              variant="outline"
              size="sm"
              onClick={downloadCode}
              className="flex items-center gap-1"
            >
              <Download className="w-4 h-4" />
              Download
            </Button>
          </div>
        </div>
      </CardHeader>
      <CardContent>
        <div className="relative">
          <div className="flex border rounded-lg overflow-hidden">
            {/* Line numbers */}
            <div className="bg-gray-50 px-2 py-3 text-sm text-gray-500 font-mono select-none border-r">
              {lineNumbers.map((num) => (
                <div key={num} className="leading-6">
                  {num}
                </div>
              ))}
            </div>
            
            {/* Code area */}
            <div className="flex-1">
              <Textarea
                value={code}
                onChange={(e) => handleCodeChange(e.target.value)}
                readOnly={readOnly}
                className="min-h-[400px] font-mono text-sm border-0 resize-none focus:ring-0 rounded-none"
                placeholder="Enter your code here..."
                style={{
                  lineHeight: '1.5',
                  tabSize: 2
                }}
              />
            </div>
          </div>
        </div>
      </CardContent>
    </Card>
  );
};

export default CodeEditor;
