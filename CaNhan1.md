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






import React, { useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Badge } from '@/components/ui/badge';
import { Wand2, RefreshCw } from 'lucide-react';
import { codeTemplates, getAllCategories, getTemplatesByCategory, CodeTemplate } from '@/lib/codeTemplates';
import CodeEditor from './CodeEditor';
import { toast } from 'sonner';

interface CodeGeneratorProps {
  onCodeGenerated?: (code: string, template: CodeTemplate) => void;
}

const CodeGenerator: React.FC<CodeGeneratorProps> = ({ onCodeGenerated }) => {
  const [selectedCategory, setSelectedCategory] = useState<string>('');
  const [selectedTemplate, setSelectedTemplate] = useState<CodeTemplate | null>(null);
  const [generatedCode, setGeneratedCode] = useState<string>('');
  const [isGenerating, setIsGenerating] = useState(false);

  const categories = getAllCategories();

  const handleCategoryChange = (category: string) => {
    setSelectedCategory(category);
    setSelectedTemplate(null);
    setGeneratedCode('');
  };

  const handleTemplateSelect = (templateId: string) => {
    const template = codeTemplates.find(t => t.id === templateId);
    if (template) {
      setSelectedTemplate(template);
      setGeneratedCode('');
    }
  };

  const generateCode = async () => {
    if (!selectedTemplate) {
      toast.error('Please select a template first');
      return;
    }

    setIsGenerating(true);
    
    // Simulate code generation process
    setTimeout(() => {
      setGeneratedCode(selectedTemplate.code);
      onCodeGenerated?.(selectedTemplate.code, selectedTemplate);
      setIsGenerating(false);
      toast.success('Code generated successfully!');
    }, 1000);
  };

  const regenerateCode = () => {
    if (selectedTemplate) {
      generateCode();
    }
  };

  const availableTemplates = selectedCategory 
    ? getTemplatesByCategory(selectedCategory)
    : [];

  return (
    <div className="space-y-6">
      <Card>
        <CardHeader>
          <CardTitle className="flex items-center gap-2">
            <Wand2 className="w-5 h-5" />
            Code Generator
          </CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          {/* Category Selection */}
          <div className="space-y-2">
            <label className="text-sm font-medium">Select Category</label>
            <Select value={selectedCategory} onValueChange={handleCategoryChange}>
              <SelectTrigger>
                <SelectValue placeholder="Choose a category" />
              </SelectTrigger>
              <SelectContent>
                {categories.map((category) => (
                  <SelectItem key={category} value={category}>
                    {category}
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
          </div>

          {/* Template Selection */}
          {selectedCategory && (
            <div className="space-y-2">
              <label className="text-sm font-medium">Select Template</label>
              <div className="grid gap-2">
                {availableTemplates.map((template) => (
                  <Card
                    key={template.id}
                    className={`cursor-pointer transition-all hover:shadow-md ${
                      selectedTemplate?.id === template.id
                        ? 'ring-2 ring-blue-500 bg-blue-50'
                        : ''
                    }`}
                    onClick={() => handleTemplateSelect(template.id)}
                  >
                    <CardContent className="p-4">
                      <div className="flex items-start justify-between">
                        <div className="space-y-1">
                          <h4 className="font-medium">{template.name}</h4>
                          <p className="text-sm text-gray-600">{template.description}</p>
                        </div>
                        <Badge variant="outline">{template.language}</Badge>
                      </div>
                    </CardContent>
                  </Card>
                ))}
              </div>
            </div>
          )}

          {/* Generate Button */}
          {selectedTemplate && (
            <div className="flex gap-2">
              <Button
                onClick={generateCode}
                disabled={isGenerating}
                className="flex items-center gap-2"
              >
                {isGenerating ? (
                  <RefreshCw className="w-4 h-4 animate-spin" />
                ) : (
                  <Wand2 className="w-4 h-4" />
                )}
                {isGenerating ? 'Generating...' : 'Generate Code'}
              </Button>
              
              {generatedCode && (
                <Button
                  variant="outline"
                  onClick={regenerateCode}
                  disabled={isGenerating}
                  className="flex items-center gap-2"
                >
                  <RefreshCw className="w-4 h-4" />
                  Regenerate
                </Button>
              )}
            </div>
          )}
        </CardContent>
      </Card>

      {/* Generated Code Display */}
      {generatedCode && (
        <CodeEditor
          initialCode={generatedCode}
          language={selectedTemplate?.language || 'javascript'}
          readOnly={false}
          onCodeChange={(code) => setGeneratedCode(code)}
        />
      )}
    </div>
  );
};

export default CodeGenerator;





import React, { useState, useEffect } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Progress } from '@/components/ui/progress';
import { Search, AlertTriangle, AlertCircle, Info, CheckCircle } from 'lucide-react';
import { runCodeReview, getReviewStats, ReviewResult } from '@/lib/reviewRules';
import CodeEditor from './CodeEditor';
import { toast } from 'sonner';

interface CodeReviewerProps {
  initialCode?: string;
}

const CodeReviewer: React.FC<CodeReviewerProps> = ({ initialCode = '' }) => {
  const [code, setCode] = useState(initialCode);
  const [reviewResults, setReviewResults] = useState<ReviewResult[]>([]);
  const [isReviewing, setIsReviewing] = useState(false);
  const [reviewStats, setReviewStats] = useState({
    total: 0,
    errors: 0,
    warnings: 0,
    info: 0
  });

  useEffect(() => {
    setCode(initialCode);
  }, [initialCode]);

  const handleReviewCode = async () => {
    if (!code.trim()) {
      toast.error('Please enter some code to review');
      return;
    }

    setIsReviewing(true);
    
    // Simulate review process
    setTimeout(() => {
      const results = runCodeReview(code);
      setReviewResults(results);
      setReviewStats(getReviewStats(results));
      setIsReviewing(false);
      
      if (results.length === 0) {
        toast.success('Great! No issues found in your code.');
      } else {
        toast.info(`Review complete: ${results.length} issues found`);
      }
    }, 1500);
  };

  const getSeverityIcon = (severity: string) => {
    switch (severity) {
      case 'error':
        return <AlertCircle className="w-4 h-4 text-red-500" />;
      case 'warning':
        return <AlertTriangle className="w-4 h-4 text-yellow-500" />;
      case 'info':
        return <Info className="w-4 h-4 text-blue-500" />;
      default:
        return <Info className="w-4 h-4" />;
    }
  };

  const getSeverityColor = (severity: string) => {
    switch (severity) {
      case 'error':
        return 'destructive';
      case 'warning':
        return 'secondary';
      case 'info':
        return 'outline';
      default:
        return 'outline';
    }
  };

  const getQualityScore = () => {
    if (reviewStats.total === 0) return 100;
    const errorWeight = reviewStats.errors * 3;
    const warningWeight = reviewStats.warnings * 1;
    const totalWeight = errorWeight + warningWeight;
    return Math.max(0, Math.min(100, 100 - totalWeight * 2));
  };

  const qualityScore = getQualityScore();

  return (
    <div className="space-y-6">
      <Card>
        <CardHeader>
          <CardTitle className="flex items-center gap-2">
            <Search className="w-5 h-5" />
            Code Reviewer
          </CardTitle>
        </CardHeader>
        <CardContent>
          <div className="space-y-4">
            <CodeEditor
              initialCode={code}
              onCodeChange={setCode}
              language="javascript"
            />
            
            <Button
              onClick={handleReviewCode}
              disabled={isReviewing}
              className="w-full"
            >
              {isReviewing ? 'Reviewing Code...' : 'Review Code'}
            </Button>
          </div>
        </CardContent>
      </Card>

      {/* Review Results */}
      {(reviewResults.length > 0 || reviewStats.total > 0) && (
        <Card>
          <CardHeader>
            <CardTitle className="flex items-center justify-between">
              <span>Review Results</span>
              <div className="flex items-center gap-2">
                <span className="text-sm font-normal">Quality Score:</span>
                <Badge variant={qualityScore >= 80 ? 'default' : qualityScore >= 60 ? 'secondary' : 'destructive'}>
                  {qualityScore}%
                </Badge>
              </div>
            </CardTitle>
          </CardHeader>
          <CardContent className="space-y-4">
            {/* Quality Score Progress */}
            <div className="space-y-2">
              <div className="flex justify-between text-sm">
                <span>Code Quality</span>
                <span>{qualityScore}%</span>
              </div>
              <Progress 
                value={qualityScore} 
                className="h-2"
              />
            </div>

            {/* Stats Summary */}
            <div className="grid grid-cols-4 gap-4">
              <div className="text-center p-3 bg-gray-50 rounded-lg">
                <div className="text-2xl font-bold">{reviewStats.total}</div>
                <div className="text-sm text-gray-600">Total Issues</div>
              </div>
              <div className="text-center p-3 bg-red-50 rounded-lg">
                <div className="text-2xl font-bold text-red-600">{reviewStats.errors}</div>
                <div className="text-sm text-gray-600">Errors</div>
              </div>
              <div className="text-center p-3 bg-yellow-50 rounded-lg">
                <div className="text-2xl font-bold text-yellow-600">{reviewStats.warnings}</div>
                <div className="text-sm text-gray-600">Warnings</div>
              </div>
              <div className="text-center p-3 bg-blue-50 rounded-lg">
                <div className="text-2xl font-bold text-blue-600">{reviewStats.info}</div>
                <div className="text-sm text-gray-600">Info</div>
              </div>
            </div>

            {/* Issues List */}
            {reviewResults.length === 0 ? (
              <Alert>
                <CheckCircle className="h-4 w-4" />
                <AlertDescription>
                  Excellent! No issues found in your code. Keep up the good work!
                </AlertDescription>
              </Alert>
            ) : (
              <div className="space-y-3">
                <h4 className="font-medium">Issues Found:</h4>
                {reviewResults.map((result, index) => (
                  <Alert key={index} className="flex items-start gap-3">
                    {getSeverityIcon(result.severity)}
                    <div className="flex-1">
                      <div className="flex items-center gap-2 mb-1">
                        <Badge variant={getSeverityColor(result.severity) as any}>
                          {result.severity.toUpperCase()}
                        </Badge>
                        <span className="text-sm text-gray-600">
                          Line {result.line}, Column {result.column}
                        </span>
                      </div>
                      <AlertDescription>
                        {result.message}
                      </AlertDescription>
                      <div className="text-xs text-gray-500 mt-1">
                        Rule: {result.rule}
                      </div>
                    </div>
                  </Alert>
                ))}
              </div>
            )}
          </div>
        </CardContent>
      </Card>
    )}
  </div>
  );
};

export default CodeReviewer;






import React, { useState, useEffect } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Badge } from '@/components/ui/badge';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from '@/components/ui/dialog';
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, AlertDialogTrigger } from '@/components/ui/alert-dialog';
import { FolderPlus, Folder, Trash2, Edit, Calendar, Code } from 'lucide-react';
import { toast } from 'sonner';

interface Project {
  id: string;
  name: string;
  description: string;
  language: string;
  code: string;
  createdAt: Date;
  updatedAt: Date;
}

interface ProjectManagerProps {
  onProjectSelect?: (project: Project) => void;
}

const ProjectManager: React.FC<ProjectManagerProps> = ({ onProjectSelect }) => {
  const [projects, setProjects] = useState<Project[]>([]);
  const [isCreateDialogOpen, setIsCreateDialogOpen] = useState(false);
  const [newProjectName, setNewProjectName] = useState('');
  const [newProjectDescription, setNewProjectDescription] = useState('');
  const [newProjectLanguage, setNewProjectLanguage] = useState('javascript');

  // Load projects from localStorage on component mount
  useEffect(() => {
    const savedProjects = localStorage.getItem('codeProjects');
    if (savedProjects) {
      const parsedProjects = JSON.parse(savedProjects).map((p: any) => ({
        ...p,
        createdAt: new Date(p.createdAt),
        updatedAt: new Date(p.updatedAt)
      }));
      setProjects(parsedProjects);
    }
  }, []);

  // Save projects to localStorage whenever projects change
  useEffect(() => {
    localStorage.setItem('codeProjects', JSON.stringify(projects));
  }, [projects]);

  const createProject = () => {
    if (!newProjectName.trim()) {
      toast.error('Project name is required');
      return;
    }

    const newProject: Project = {
      id: Date.now().toString(),
      name: newProjectName.trim(),
      description: newProjectDescription.trim(),
      language: newProjectLanguage,
      code: '',
      createdAt: new Date(),
      updatedAt: new Date()
    };

    setProjects(prev => [newProject, ...prev]);
    setNewProjectName('');
    setNewProjectDescription('');
    setNewProjectLanguage('javascript');
    setIsCreateDialogOpen(false);
    toast.success('Project created successfully!');
  };

  const deleteProject = (projectId: string) => {
    setProjects(prev => prev.filter(p => p.id !== projectId));
    toast.success('Project deleted successfully!');
  };

  const selectProject = (project: Project) => {
    onProjectSelect?.(project);
    toast.success(`Opened project: ${project.name}`);
  };

  const updateProject = (projectId: string, updates: Partial<Project>) => {
    setProjects(prev => prev.map(p => 
      p.id === projectId 
        ? { ...p, ...updates, updatedAt: new Date() }
        : p
    ));
  };

  const getLanguageColor = (language: string) => {
    const colors: { [key: string]: string } = {
      javascript: 'bg-yellow-100 text-yellow-800',
      typescript: 'bg-blue-100 text-blue-800',
      python: 'bg-green-100 text-green-800',
      java: 'bg-red-100 text-red-800',
      sql: 'bg-purple-100 text-purple-800',
      html: 'bg-orange-100 text-orange-800',
      css: 'bg-pink-100 text-pink-800'
    };
    return colors[language] || 'bg-gray-100 text-gray-800';
  };

  const formatDate = (date: Date) => {
    return new Intl.DateTimeFormat('en-US', {
      month: 'short',
      day: 'numeric',
      year: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    }).format(date);
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle className="flex items-center gap-2">
            <Folder className="w-5 h-5" />
            Project Manager
          </CardTitle>
          <Dialog open={isCreateDialogOpen} onOpenChange={setIsCreateDialogOpen}>
            <DialogTrigger asChild>
              <Button className="flex items-center gap-2">
                <FolderPlus className="w-4 h-4" />
                New Project
              </Button>
            </DialogTrigger>
            <DialogContent>
              <DialogHeader>
                <DialogTitle>Create New Project</DialogTitle>
              </DialogHeader>
              <div className="space-y-4">
                <div className="space-y-2">
                  <label className="text-sm font-medium">Project Name</label>
                  <Input
                    value={newProjectName}
                    onChange={(e) => setNewProjectName(e.target.value)}
                    placeholder="Enter project name"
                  />
                </div>
                <div className="space-y-2">
                  <label className="text-sm font-medium">Description</label>
                  <Input
                    value={newProjectDescription}
                    onChange={(e) => setNewProjectDescription(e.target.value)}
                    placeholder="Enter project description"
                  />
                </div>
                <div className="space-y-2">
                  <label className="text-sm font-medium">Language</label>
                  <select
                    value={newProjectLanguage}
                    onChange={(e) => setNewProjectLanguage(e.target.value)}
                    className="w-full p-2 border rounded-md"
                  >
                    <option value="javascript">JavaScript</option>
                    <option value="typescript">TypeScript</option>
                    <option value="python">Python</option>
                    <option value="java">Java</option>
                    <option value="sql">SQL</option>
                    <option value="html">HTML</option>
                    <option value="css">CSS</option>
                  </select>
                </div>
                <div className="flex justify-end gap-2">
                  <Button variant="outline" onClick={() => setIsCreateDialogOpen(false)}>
                    Cancel
                  </Button>
                  <Button onClick={createProject}>
                    Create Project
                  </Button>
                </div>
              </div>
            </DialogContent>
          </Dialog>
        </div>
      </CardHeader>
      <CardContent>
        {projects.length === 0 ? (
          <div className="text-center py-8 text-gray-500">
            <Folder className="w-12 h-12 mx-auto mb-4 opacity-50" />
            <p>No projects yet. Create your first project to get started!</p>
          </div>
        ) : (
          <div className="space-y-3">
            {projects.map((project) => (
              <Card
                key={project.id}
                className="cursor-pointer transition-all hover:shadow-md hover:border-blue-300"
                onClick={() => selectProject(project)}
              >
                <CardContent className="p-4">
                  <div className="flex items-start justify-between">
                    <div className="flex-1 space-y-2">
                      <div className="flex items-center gap-2">
                        <h4 className="font-medium">{project.name}</h4>
                        <Badge className={getLanguageColor(project.language)}>
                          {project.language}
                        </Badge>
                      </div>
                      
                      {project.description && (
                        <p className="text-sm text-gray-600">{project.description}</p>
                      )}
                      
                      <div className="flex items-center gap-4 text-xs text-gray-500">
                        <div className="flex items-center gap-1">
                          <Calendar className="w-3 h-3" />
                          Created: {formatDate(project.createdAt)}
                        </div>
                        <div className="flex items-center gap-1">
                          <Edit className="w-3 h-3" />
                          Updated: {formatDate(project.updatedAt)}
                        </div>
                        <div className="flex items-center gap-1">
                          <Code className="w-3 h-3" />
                          {project.code.length} chars
                        </div>
                      </div>
                    </div>
                    
                    <AlertDialog>
                      <AlertDialogTrigger asChild>
                        <Button
                          variant="outline"
                          size="sm"
                          className="ml-2"
                          onClick={(e) => e.stopPropagation()}
                        >
                          <Trash2 className="w-4 h-4" />
                        </Button>
                      </AlertDialogTrigger>
                      <AlertDialogContent>
                        <AlertDialogHeader>
                          <AlertDialogTitle>Delete Project</AlertDialogTitle>
                          <AlertDialogDescription>
                            Are you sure you want to delete "{project.name}"? This action cannot be undone.
                          </AlertDialogDescription>
                        </AlertDialogHeader>
                        <AlertDialogFooter>
                          <AlertDialogCancel>Cancel</AlertDialogCancel>
                          <AlertDialogAction
                            onClick={() => deleteProject(project.id)}
                            className="bg-red-600 hover:bg-red-700"
                          >
                            Delete
                          </AlertDialogAction>
                        </AlertDialogFooter>
                      </AlertDialogContent>
                    </AlertDialog>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        )}
      </CardContent>
    </Card>
  );
};

export default ProjectManager;





import React, { useState, useEffect } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Progress } from '@/components/ui/progress';
import { History, TrendingUp, AlertTriangle, CheckCircle, Calendar, Trash2 } from 'lucide-react';
import { ReviewResult } from '@/lib/reviewRules';

interface ReviewRecord {
  id: string;
  projectName: string;
  timestamp: Date;
  results: ReviewResult[];
  qualityScore: number;
  codeLength: number;
}

interface ReviewHistoryProps {
  currentReview?: {
    projectName: string;
    results: ReviewResult[];
    qualityScore: number;
    codeLength: number;
  };
}

const ReviewHistory: React.FC<ReviewHistoryProps> = ({ currentReview }) => {
  const [reviewHistory, setReviewHistory] = useState<ReviewRecord[]>([]);

  // Load review history from localStorage
  useEffect(() => {
    const savedHistory = localStorage.getItem('reviewHistory');
    if (savedHistory) {
      const parsedHistory = JSON.parse(savedHistory).map((record: any) => ({
        ...record,
        timestamp: new Date(record.timestamp)
      }));
      setReviewHistory(parsedHistory);
    }
  }, []);

  // Save current review to history
  useEffect(() => {
    if (currentReview) {
      const newRecord: ReviewRecord = {
        id: Date.now().toString(),
        projectName: currentReview.projectName,
        timestamp: new Date(),
        results: currentReview.results,
        qualityScore: currentReview.qualityScore,
        codeLength: currentReview.codeLength
      };

      const updatedHistory = [newRecord, ...reviewHistory].slice(0, 50); // Keep last 50 reviews
      setReviewHistory(updatedHistory);
      localStorage.setItem('reviewHistory', JSON.stringify(updatedHistory));
    }
  }, [currentReview]);

  const deleteRecord = (recordId: string) => {
    const updatedHistory = reviewHistory.filter(record => record.id !== recordId);
    setReviewHistory(updatedHistory);
    localStorage.setItem('reviewHistory', JSON.stringify(updatedHistory));
  };

  const clearAllHistory = () => {
    setReviewHistory([]);
    localStorage.removeItem('reviewHistory');
  };

  const getQualityBadgeVariant = (score: number) => {
    if (score >= 80) return 'default';
    if (score >= 60) return 'secondary';
    return 'destructive';
  };

  const formatDate = (date: Date) => {
    return new Intl.DateTimeFormat('en-US', {
      month: 'short',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    }).format(date);
  };

  const getAverageQualityScore = () => {
    if (reviewHistory.length === 0) return 0;
    const sum = reviewHistory.reduce((acc, record) => acc + record.qualityScore, 0);
    return Math.round(sum / reviewHistory.length);
  };

  const getTotalIssuesFound = () => {
    return reviewHistory.reduce((acc, record) => acc + record.results.length, 0);
  };

  const getRecentTrend = () => {
    if (reviewHistory.length < 2) return 'stable';
    const recent = reviewHistory.slice(0, 5);
    const older = reviewHistory.slice(5, 10);
    
    if (recent.length === 0 || older.length === 0) return 'stable';
    
    const recentAvg = recent.reduce((acc, r) => acc + r.qualityScore, 0) / recent.length;
    const olderAvg = older.reduce((acc, r) => acc + r.qualityScore, 0) / older.length;
    
    if (recentAvg > olderAvg + 5) return 'improving';
    if (recentAvg < olderAvg - 5) return 'declining';
    return 'stable';
  };

  const trend = getRecentTrend();

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle className="flex items-center gap-2">
            <History className="w-5 h-5" />
            Review History
          </CardTitle>
          {reviewHistory.length > 0 && (
            <Button
              variant="outline"
              size="sm"
              onClick={clearAllHistory}
              className="text-red-600 hover:text-red-700"
            >
              <Trash2 className="w-4 h-4 mr-1" />
              Clear All
            </Button>
          )}
        </div>
      </CardHeader>
      <CardContent>
        {reviewHistory.length === 0 ? (
          <div className="text-center py-8 text-gray-500">
            <History className="w-12 h-12 mx-auto mb-4 opacity-50" />
            <p>No review history yet. Start reviewing code to see your progress!</p>
          </div>
        ) : (
          <div className="space-y-6">
            {/* Statistics Overview */}
            <div className="grid grid-cols-3 gap-4">
              <div className="text-center p-4 bg-blue-50 rounded-lg">
                <div className="text-2xl font-bold text-blue-600">{reviewHistory.length}</div>
                <div className="text-sm text-gray-600">Total Reviews</div>
              </div>
              <div className="text-center p-4 bg-green-50 rounded-lg">
                <div className="text-2xl font-bold text-green-600">{getAverageQualityScore()}%</div>
                <div className="text-sm text-gray-600">Avg Quality</div>
              </div>
              <div className="text-center p-4 bg-orange-50 rounded-lg">
                <div className="text-2xl font-bold text-orange-600">{getTotalIssuesFound()}</div>
                <div className="text-sm text-gray-600">Issues Found</div>
              </div>
            </div>

            {/* Trend Indicator */}
            <div className="flex items-center gap-2 p-3 bg-gray-50 rounded-lg">
              <TrendingUp className={`w-5 h-5 ${
                trend === 'improving' ? 'text-green-500' : 
                trend === 'declining' ? 'text-red-500' : 'text-gray-500'
              }`} />
              <span className="font-medium">Recent Trend:</span>
              <Badge variant={
                trend === 'improving' ? 'default' : 
                trend === 'declining' ? 'destructive' : 'secondary'
              }>
                {trend === 'improving' ? 'Improving' : 
                 trend === 'declining' ? 'Declining' : 'Stable'}
              </Badge>
            </div>

            {/* Review Records */}
            <div className="space-y-3">
              <h4 className="font-medium">Recent Reviews</h4>
              {reviewHistory.slice(0, 10).map((record) => (
                <Card key={record.id} className="relative">
                  <CardContent className="p-4">
                    <div className="flex items-start justify-between">
                      <div className="flex-1 space-y-2">
                        <div className="flex items-center gap-2">
                          <h5 className="font-medium">{record.projectName}</h5>
                          <Badge variant={getQualityBadgeVariant(record.qualityScore)}>
                            {record.qualityScore}%
                          </Badge>
                        </div>
                        
                        <div className="flex items-center gap-4 text-sm text-gray-600">
                          <div className="flex items-center gap-1">
                            <Calendar className="w-3 h-3" />
                            {formatDate(record.timestamp)}
                          </div>
                          <div className="flex items-center gap-1">
                            {record.results.length === 0 ? (
                              <CheckCircle className="w-3 h-3 text-green-500" />
                            ) : (
                              <AlertTriangle className="w-3 h-3 text-yellow-500" />
                            )}
                            {record.results.length} issues
                          </div>
                          <div>
                            {record.codeLength} characters
                          </div>
                        </div>

                        <div className="space-y-1">
                          <div className="flex justify-between text-xs">
                            <span>Quality Score</span>
                            <span>{record.qualityScore}%</span>
                          </div>
                          <Progress value={record.qualityScore} className="h-1" />
                        </div>

                        {record.results.length > 0 && (
                          <div className="text-xs text-gray-500">
                            Issues: {record.results.filter(r => r.severity === 'error').length} errors, {' '}
                            {record.results.filter(r => r.severity === 'warning').length} warnings
                          </div>
                        )}
                      </div>
                      
                      <Button
                        variant="ghost"
                        size="sm"
                        onClick={() => deleteRecord(record.id)}
                        className="text-gray-400 hover:text-red-600"
                      >
                        <Trash2 className="w-4 h-4" />
                      </Button>
                    </div>
                  </CardContent>
                </Card>
              ))}
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
};

export default ReviewHistory;
