[🏠](README.md) › [Node.js](nodejs.md) › Projects


# Projects

- [Structure](#structure)
- [Dependencies](#dependencies)
- [Hapi.js](#hapijs)
- [Documentation](#documentation)


## Structure  
- Each project should resemble the following structure.
- A hapi server will also have a `server.js` or `manifest.json`.  

**simple module**  
```
.git
├── lib
│   ├── core.js
│   ├── index.js
│   └── shell.js
├── node_modules
├── test
│   └── index.js
.gitignore
gitlab-ci.yml
npm-shrinkwrap.json
package.json
README.md
```

## Dependencies  
- Not all dependencies will be semver compliant, therefore lock down exact versions of dependencies & transitive dependencies (dependencies of dependencies) by running `npm shrinkwrap --dev`. This will create or update a `npm-shrinkwrap.json` file in the root directory of the project.

**gotchas**  
- If a dependency has been renamed, the shrinkwrap will fail!

**Additional reading**  
- [Why semver ranges are literally the worst](https://medium.com/@kentcdodds/why-semver-ranges-are-literally-the-worst-817cdcb09277#.fyvskznub)
- [npm-shrinkwrap: Lock down dependency versions](https://docs.npmjs.com/cli/shrinkwrap)


### Hapi.js
- Hapi.js provides a plugin architecture for managing application complexity.  
- Project specific business logic is located within the `lib` folder, where each subdirectory is a plugin.  
- Plugins that are project agnostic should be a dependency (located in `node_modules`) and be their own project, prefixed with `plugin-`.  
- Plugins will at times depend on other plugins; use `server.dependency` to prevent any race conditions.  

```javascript
exports.register = (server, options, next) => {
  ...
  server.dependency('plugin-auth-token', (serverDep, nextDep) => {
    // dependency has been loaded, place module code here
  });
  ...
});
```

- Prefer configuration over code, service initialization is performed by environment aware `manifest.json` file + `good`, `confidence` hapi plugins.  


**Additional reading**  
- [hapiDay Oakland 2014 | John Brett: hapi Adoption at D4H](https://www.youtube.com/watch?v=x7nq3Q0xrEI)


### Documentation
- Every repo should have a well documented README.md file.
- Each README.md file should follow a similar pattern:
  - Project Title (h1)
  - Succinct Project description or tagline (quote)
  - Badge for Gitlab ci build status (img)
  - Expanded project description (p)
  - API Reference (h2) with current version
  - Links to api methods
  - API methods

```markdown
# courier

> fault tolerant service requests

...
```

**Additional reading**  
- [Readme Driven Development](http://tom.preston-werner.com/2010/08/23/readme-driven-development.html)
