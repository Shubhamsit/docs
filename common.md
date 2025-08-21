### Having error in process.env , when configuring dotenv

- install

```
npm install --save-dev @types/node
```

- Need to add "types":["node"] in compiler options

```json
{
  "compilerOptions": {
        ... more properties
    "types": ["node"],
    "strict": true
        ... more properties 
  }
}

```

- Use `import 'dotenv/config'` in files 

