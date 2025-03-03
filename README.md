# Git-Analyze

A powerful GitHub repository analysis tool that supports code knowledge graph construction, semantic code search, and code structure visualization.

## Core Features

- Support code analysis from GitHub URL or local directory
- Intelligent code dependency analysis with automatic file tracking
- Generate code knowledge graphs showing code structure and relationships
- Support code token consumption estimation
- Support binary file filtering and size limits
- Support custom domain mapping
- Intelligent code search based on knowledge graphs
- Support semantic code understanding and relationship analysis
- Automatically build code knowledge base with contextual relevance search
- Free and open source, no API key required

## Workflow

### 1. Code Acquisition

- Support cloning code from GitHub URL
- Support analyzing local directories
- Automatic temporary file cleanup

### 2. Intelligent Knowledge Graph

The system automatically analyzes code structure and dependencies to build a knowledge graph:

- Automatic identification of code entities (classes, functions, variables, etc.)
- Analysis of entity relationships (calls, inheritance, implementation)
- Construction of semantic code knowledge network
- Support for relevance-based search using the graph
- Provide visualization of the knowledge graph

### 3. Dependency Analysis

```151:194:src/core/scanner.ts
  // [Analyze dependencies as needed]: Analyze dependent files
  protected async analyzeDependencies(
    content: string,
    filePath: string,
    basePath: string
  ): Promise<string[]> {
    const dependencies: string[] = [];
    // Match import paths. Example: import { Button } from '@/components/Button'
    const importRegex = /(?:import|from)\s+['"]([^'"]+)['"]/g;

    // Remove multi-line comments
    const contentWithoutComments = content.replace(/\/\*[\s\S]*?\*\//g, "");
    const lines = contentWithoutComments
      .split("\n")
      .filter((line) => {
        const trimmed = line.trim();
        return trimmed && !trimmed.startsWith("//");
      })
      .join("\n");

    // Match import paths
    let match;
    // Iterate through each line, matching import paths
    while ((match = importRegex.exec(lines)) !== null) {
      // Get import path. Example: import { Button } from '@/components/Button'
      const importPath = match[1];
      // Get current file path. Example: src/components/Button/index.ts
      const currentDir = dirname(filePath);

      // Find import path. Example: src/components/Button/index.ts
      const resolvedPath = await this.findModuleFile(
        importPath,
        currentDir,
        basePath
      );
      // If import path exists and not in dependencies list, add it
      if (resolvedPath && !dependencies.includes(resolvedPath)) {
        dependencies.push(resolvedPath);
      }
    }

    // Return dependencies list. Example: ['src/components/Button/index.ts', 'src/components/Input/index.ts']
    return dependencies;
  }
```

### 4. Code Analysis

```71:90:src/core/codeAnalyzer.ts
  public analyzeCode(filePath: string, sourceCode: string): void {
    if (!filePath) {
      throw new Error('File path cannot be undefined');
    }
    this.currentFile = filePath;
    try {
      console.log(`[CodeAnalyzer] Processing file: ${filePath}`);

      const tree = this.parser.parse(sourceCode);
      console.log(`[CodeAnalyzer] AST generated for ${filePath}`);

      this.visitNode(tree.rootNode);

      console.log(`[CodeAnalyzer] Analysis complete for ${filePath}`);
      console.log(`[CodeAnalyzer] Found ${this.codeElements.length} nodes`);
      console.log(`[CodeAnalyzer] Found ${this.relations.length} relationships`);
    } catch (error) {
      console.error(`[CodeAnalyzer] Error analyzing file ${filePath}:`, error);
    }
  }
```

### 5. Knowledge Graph Generation

```382:431:src/core/codeAnalyzer.ts
  public getKnowledgeGraph(): KnowledgeGraph {
    console.log(`[Debug] Generating knowledge graph:`, {
      totalElements: this.codeElements.length,
      totalRelations: this.relations.length
    });

    // 1. First convert nodes, add implementation field
    const nodes: KnowledgeNode[] = this.codeElements.map(element => ({
      id: element.id!,
      name: element.name,
      type: element.type,
      filePath: element.filePath,
      location: element.location,
      implementation: element.implementation || '' // Add implementation field
    }));

    // 2. Validate all relationships
    const validRelations = this.relations.filter(relation => {
      const sourceExists = this.codeElements.some(e => e.id === relation.sourceId);
      const targetExists = this.codeElements.some(e => e.id === relation.targetId);

      if (!sourceExists || !targetExists) {
        console.warn(`[Warning] Invalid relation:`, {
          source: relation.sourceId,
          target: relation.targetId,
          type: relation.type,
          sourceExists,
          targetExists
        });
        return false;
      }
      return true;
    });

    // 3. Convert relationships
    const edges: KnowledgeEdge[] = validRelations.map(relation => ({
      source: relation.sourceId,
      target: relation.targetId,
      type: relation.type,
      properties: {}
    }));

    console.log(`[Debug] Knowledge graph generated:`, {
      nodes: nodes.length,
      edges: edges.length,
      relationTypes: new Set(edges.map(e => e.type))
    });

    return { nodes, edges };
  }
```

## Usage Example

```typescript
import { GitIngest } from "git-analyze";

// Create instance
const analyzer = new GitIngest({
  tempDir: "temp",
  defaultMaxFileSize: 1024 * 1024, // 1MB
  defaultPatterns: {
    include: ["**/*"],
    exclude: ["**/node_modules/**"],
  },
});

// Analyze from GitHub
const result = await analyzer.analyzeFromUrl(
  "https://github.com/username/repo",
  {
    branch: "main",
    targetPaths: ["src/"],
  }
);

// Analyze from local directory
const result = await analyzer.analyzeFromDirectory("./my-project", {
  maxFileSize: 2 * 1024 * 1024,
  includePatterns: ["src/**/*.ts"],
});

// Analysis results
console.log(result.metadata); // Project metadata
console.log(result.fileTree); // File tree structure
console.log(result.sizeTree); // Size tree structure
console.log(result.codeAnalysis); // Code analysis results

// Get knowledge graph
const graph = result.knowledgeGraph;

// Search related code
const searchResult = await analyzer.searchRelatedCode("UserService", {
  maxResults: 10,
  includeContext: true,
});

// Get code relationships
const relations = await analyzer.getCodeRelations("src/services/user.ts");
```

## Configuration Options

### GitIngestConfig

```75:94:src/types/index.ts
export interface GitIngestConfig {
  // Temporary directory name for storing cloned repositories
  tempDir?: string;
  /* Maximum file size for scanning */
  defaultMaxFileSize?: number;
  /* File patterns */
  defaultPatterns?: {
    /* Files/directories to include */
    include?: string[];
    /* Files/directories to exclude from scanning */
    exclude?: string[];
  };
  /* Keep cloned repositories */
  keepTempFiles?: boolean;
  /* Custom domain mapping */
  customDomainMap?: {
    targetDomain: string;
    originalDomain: string;
  };
}
```

### AnalyzeOptions

```1:14:src/types/index.ts
export interface AnalyzeOptions {
  // Maximum file size
  maxFileSize?: number;
  // Include file patterns
  includePatterns?: string[];
  // Exclude file patterns
  excludePatterns?: string[];
  // Target file paths
  targetPaths?: string[];
  // Branch
  branch?: string;
  // Commit
  commit?: string;
}
```

## Token Estimation Algorithm

The tool uses a smart algorithm to estimate code token consumption:

```4:18:src/utils/index.ts
export function estimateTokens(text: string): number {
  // 1. Calculate Chinese character count
  const chineseChars = (text.match(/[\u4e00-\u9fff]/g) || []).length;

  // 2. Calculate English word count (including numbers and punctuation)
  const otherChars = text.length - chineseChars;

  // 3. Calculate total tokens:
  // - Chinese characters typically have a 1:1 or 1:2 ratio, conservatively using 2
  // - Other characters use a 1:0.25 ratio
  const estimatedTokens = chineseChars * 2 + Math.ceil(otherChars / 4);

  // 4. Add 10% safety margin
  return Math.ceil(estimatedTokens * 1.1);
}
```

## Installation

```bash
npm install git-analyze
```

## License

MIT

## Tech Stack

- TypeScript
- tree-sitter (AST parsing)
- glob (file matching)
- Knowledge graph algorithms

## Notes

1. Temporary files are not saved by default, can be retained via `keepTempFiles` configuration
2. Binary files and node_modules are filtered by default
3. Supported code file types: .ts, .tsx, .js, .jsx, .vue
4. Token estimates include a 10% safety margin
