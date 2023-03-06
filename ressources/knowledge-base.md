# Table of Contents

<p><a href="#git">Git</a></p>
<p>--- <a href="#config-file">Config file</a></p>
<p>--- <a href="#initialize-local-repository">Initialize local repository</a></p>
<p>--- <a href="#git-version">Get git version</a></p>
<p>--- <a href="#reset-staged-files">Reset staged files</a></p>
<p>--- <a href="#handle-commits">Handle local changes and commits</a></p>
<p>--- <a href="#dealing-with-branches">Dealing with branches</a></p>
<p>--- <a href="#dealing-with-remote-repo">Dealing with a remote repository</a></p>
<p>--- <a href="#miscellaneous">Miscellaneous</a></p>
<p><a href="#deployment">Deployment</a></p>
<p>--- <a href="#deploy-react-app-to-gh-pages">Deploy react app to gh-pages</a></p>
<p><a href="#react">React</a></p>
<p>--- <a href="#react-installation">React installation</a></p>
<p>--- <a href="#fetch-with-axios">Fetch with axios</a></p>
<p>--- <a href="#react-use-reducer">Use reducer</a></p>
<p><a href="#Node-js">Node js</a></p>
<p>--- <a href="#node-js-import-json">Standard import json</a></p>
<p>--- <a href="#node-js-import-keyword">Node js with the import-keyword</a></p>
<p>--- <a href="#node-js-import-json-from-js-file">Importing json from js file with import</a></p>


<h1 align="center">Knowledge-base</h1>

<h1 id="git" align="center">GIT</h1>

<h2 id="config-file" style="color:#37d">Config file</h2>

List config file (global)

    git config --global -l

List config file (local, in .git folder)

    git config --local -l

Delete variable (example: user name) of config file (local)

    git config --local --unset user.name

Set shortcut for command

    git config --global alias.s status // git s would print the status
    git config --global alias.ac "git add . && git commit -m" // git ac "message" would add and commit

Set "main" as default name for new repositories

    git config -- global init.defaultBranch main

<h2 id="initialize-local-repository" style="color:#37d">Initialize local repository</h2>

Initialize with branch main instead of master

    git init -b main

<h2 id="git-version" style="color:#37d">Get git version</h2>

    git --version

<h2 id="reset-staged-files" style="color:#37d">Reset staged files</h2>

Remove one file from stage area

    git reset <filename>

Remove all files from stage area

    git reset


<h2 id="handle-commits" style="color:#37d">Handle local changes and commits</h2>

## Undo local changes (back to state of last commit)

    git restore .

or (before git version 2.23)

    git checkout -- .

Undo but keep changes in a special area and use them later <br/>
with "git stash pop" or remove them with "git stash drop"

    git stash

## Restore an old version locally (jump to a specific commit)
Back to last commit with HEAD. Second to last commit with HEAD~, older ones with HEAD~1 etc.<br/>
or use a commit hash instead of HEAD

Soft reset (changes are kept inside own workspace)

    git reset HEAD~1

Hard reset (all changes are discarded)

    git reset --hard HEAD~1

or

    git restore -s HEAD

or (before git version 2.23)

    1. git checkout HEAD

Since a detached head is created, to continue with other commits you need to connect to a branch again

    2. git switch -

or

    2. git switch -c <new branch name>

## Take back a specific commit

Starts editor which needs a commit message<br/>
only the targeted commit is removed, not following commits

    git revert HEAD

Take back the last x commits (example reverts last 3 commits)
Use flag -n and manually commit with a message otherwise a commit message is necessary for each commit

    1. git revert HEAD~2..HEAD -n

    2. git commit -m "Reverted last three commits"

## Change commit messages (locally)
Change last message

    git commit --amend -m "new message"

Change random commit message

    1. git rebase --interactive origin <branch>
    2. reword <commit> changed <message> // Inside the editor
    3. // confirm new name in file that opened automatically

## Remove pushed commit from github

    1. git revert <commit-id>
    2. // change or close commit message in file that opened automatically
    3. git push

<h2 id="dealing-with-branches" style="color:#37d">Dealing with branches</h2>

Creating a new branch

    git branch <name>

Creating a new branch and switching to the new branch

    git checkout -b <name>

Rename branch

    git branch -m <new name>

Delete branch locally

    git branch -d <name>

Delete branch remotely

    git push origin --delete <name>

Push to non existing remote branch

    git push -u origin <branch name>

Push all local branches

    git push --all origin

<h2 id="dealing-with-remote-repo" style="color:#37d">Dealing with a remote repository</h2>

Connecting local repo to empty remote repo

    git remote add origin <url>

## Add local repository with different history to new remote reposito

## Delete file from git but keep it locally

    1. git rm --cached <filename>
    2. git commit -m "removed file"

<h2 id="miscellaneous" style="color:#37d">Miscellaneous</h2>

## Removing node modules from remote

    git rm -r --cached node_modules

    git commit -m "Removed node_modules"

    git push origin master

## Ignoring whitespaces for merging

    git merge -X ignore-all-space
    
## Cancel a current merge conflict

    git merge --abort

<h1 id="deployment" align="center">DEPLOYMENT</h1>

<h2 id="deploy-react-app-to-gh-pages" style="color:#37d">Deploy React app to gh-pages</h2>

    1. npm install gh-pages --save-dev

In package.json (above "name")

    2. "homepage": "http://<github name>.github.io/<repository name>"

In package.json (under "scripts")

    3. "predeploy": "npm run build",
    4. "deploy": "gh-pages -d build"

Add and commit changes and start deploying with:

    5. npm run deploy
<h1 id="react" align="center">REACT</h1>

<h2 id="react-installation" style="color:#37d">Installation</h2>

With vite

    1. npm create vite
    2. // Enter project name
    3. // Choose system vanilla, react, etc.
    4. npm install
    5. npm run dev

<h2 id="fetch-with-axios" style="color:#37d">Fetch with axios</h2>

Advantage of axios: no converting to json needed

    npm install axios
    
```js

import axios from "axios";

(async ()=>{
   const response = await axios.get(url).data
})();
 
```

<h2 id="react-use-reducer" style="color:#37d">useReducer</h2>

    import {useReducer} from "react";

```js

function reducer(state, action){
    switch (action.type){
        case "increment":
            return {count: state.count + 1}
        case "decrement":
            return {count: state.count - 1}
        default:
            return state
    }
}

const [state, dispatch] = useReducer(reducer, {count: 0})

function increment(){
    dispatch({type: "increment"})
}

function decrement(){
    dispatch({type: "decrement"})
}

return (
    <button onClick={increment}>Increment</button>
    <button onClick={decrement}>Decrement</button>
)
```

<h1 id="node-js" align="center">NODE JS</h1>

<h2 id="node-js-import-json" style="color:#37d">Importing json</h2>

    const user = require("./data.json")

or

    const user = require("./data")

From Node js version 16.18.1 and up (only default export supported)

<h2 id="node-js-import-keyword" style="color:#37d">Importing by using the keyword import</h2>

In package.json

    1. "type": "module"

In Server js

    2. import data from "./data.json" assert {type: "json"}


<h2 id="node-js-import-json-from-js-file" style="color:#37d">Importing json from js file with import</h2>


In data file

    1. export default [
        {...}, {...}, {...}
    ]

In package.json

    2. "type": "module"

In server.js

    3. import data from "./data.js"
    
# REDUX TOOLKIT

## Folder structure
```js
app
    -store.js
features
    -dataSlice.js
```

Inside app/store.js
```js
import {configureStore} from "@reduxjs/toolkit";
import languageDataReducer from "../features/languageDataSlice";

export const store = configureStore({
    reducer: {
        languageData:languageDataReducer
    } // Place for all the slices
})
```
Inside features/dataSlice.js
```js
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
    value: {fData:{}, selectedLanguage: ""}
}

export const languageDataSlice = createSlice({
    name: "languageData",
    initialState,
    reducers: {
        updateLanguageData: (state, action) => {
            state.value.fData = action.payload
            state.value.selectedLanguage = action.payload.language
        }
    }
})

export const {updateLanguageData} = languageDataSlice.actions;
export default languageDataSlice.reducer;
```

Inside a file where the store is read/updated
```js
import {useSelector, useDispatch} from 'react-redux'; 
import { updateLanguageData } from '../../features/languageDataSlice'; 

const languageData = useSelector((state) => state.languageData.value); // Get value from the state
const dispatch = useDispatch();
dispatch(updateLanguageData(value)); // Change the state
```