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
