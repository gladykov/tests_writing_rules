# Opinionated rules to test writing (in TS but not only)

## Keep const together, and let together

Bad:

```
const environment = envConfig.environments.staging;
const possibleGitProviders = ['gitlab', 'github', 'bitbucket'];

let page: Login;
let spacePage: Space;
let pageTitle: string;
let recordsWithData: Array<Record<string, string>>;

const CommonFn = new Common();
const testedComponent = 'myComponent';
```

Good:

```
const environment = envConfig.environments.staging;
const possibleGitProviders = ['gitlab', 'github', 'bitbucket'];
const CommonFn = new Common();
const testedComponent = 'myComponent';

let page: Login;
let spacePage: Space;
let pageTitle: string;
let recordsWithData: Array<Record<string, string>>;
```

## Prepare all data at the beginning of test

Bad:

```
pageTitle = CommonFn.pageTitle;

const uiDialog = await createNewPage.activateComponent(testedComponent, 'click');

const fixture = fixtures.get(1);

await uiDialog.switchToTab('RemoteURL');

const gitRepo = new GitRepo(gitProvider, envConfig.gitServices);

await uiDialog.insertURL(gitRepo.fileUrl(fixture.input().name, false));
```

Good:
```
pageTitle = CommonFn.pageTitle;
const fixture = fixtures.get(1);
const gitRepo = new GitRepo(gitProvider, envConfig.gitServices);

const uiDialog = await createNewPage.activateComponent(testedComponent, 'click');
await uiDialog.switchToTab('RemoteURL');
await uiDialog.insertURL(gitRepo.fileUrl(fixture.input().name, false));
```

## If you need to prepare more data during test flow, group them together

Bad:
```
(...)

await uiDialog.switchToTab('RemoteURL');
expect(await confluencePage.getText(testedComponent)).toBe(fixture.output().content);

const gitRepo = new GitRepo(gitProvider, envConfig.gitServices);

await uiDialog.insertURL(gitRepo.fileUrl(tabsInputOutputFiles[2].fixture.input().name, false));

selectedTab = faker.helpers.arrayElement(tabsInputOutputFiles);

await createNewPage.insertSaveComponent('insert', uiDialog, testedComponent);
```

Good:
```
await uiDialog.switchToTab('RemoteURL');
expect(await confluencePage.getText(testedComponent)).toBe(fixture.output().content);

// Data preparation step later in the game
const gitRepo = new GitRepo(gitProvider, envConfig.gitServices);
selectedTab = faker.helpers.arrayElement(tabsInputOutputFiles);

await uiDialog.insertURL(gitRepo.fileUrl(tabsInputOutputFiles[2].fixture.input().name, false));
await createNewPage.insertSaveComponent('insert', uiDialog, testedComponent);
```



## Add line break after assertion

*For complex scenarios I use more than one expect in the test, not to lose time*

Bad:

```
await uiDialog.editURL(gitRepo.fileUrl(url));
expect(await confluencePage.getText(testedComponent)).toBe(expected);
await uiDialog.useAndCheckPreview(fixture.output().content, false, testedComponent);
await createNewPage.insertSaveComponent('save', uiDialog, testedComponent);
expect(await confluencePage.getText(testedComponent)).toBe(expected);
const confluencePage = await createNewPage.publish();
```

Good:
```
await uiDialog.editURL(url));
expect(await confluencePage.getText(testedComponent)).toBe(expected);

await uiDialog.useAndCheckPreview(fixture.output().content, false, testedComponent);
await createNewPage.insertSaveComponent('save', uiDialog, testedComponent);
expect(await confluencePage.getText(testedComponent)).toBe(expected);

const confluencePage = await createNewPage.publish();
```


## Add line break when switching between data preparation and step execution

Bad:
```
await confluencePage.editPage();
uiDialog = await createNewPage.editComponent('RemoteURL', testedComponent);
gitRepo = new GitRepo(secondGitProvider, envConfig.gitServices);
await uiDialog.editURL(gitRepo.fileUrl(fixtures.get(2).input().name, false));
```


Good:
```
await confluencePage.editPage();
uiDialog = await createNewPage.editComponent('RemoteURL', testedComponent);

gitRepo = new GitRepo(secondGitProvider, envConfig.gitServices);

await uiDialog.editURL(gitRepo.fileUrl(fixtures.get(2).input().name, false));
```

## Prepare new data after assertion

Sometimes it is not possible to prepare all data in the beginning of the test. In that case, prepare new data after assertion

Bad:
```
const fixture1 = fixtures.get(1);

await uiDialog.insertURL(gitRepo.fileUrl(fixture1.input().name, false));

const fixture2 = fixtures.get(2);

expect(await confluencePage.getText(testedComponent)).toBe(expectedOutput);

await uiDialog.insertURL(gitRepo.fileUrl(fixture2.input().name, false));

```


Good:
```
const fixture1 = fixtures.get(1);

await uiDialog.insertURL(gitRepo.fileUrl(fixture1.input().name, false));
expect(await confluencePage.getText(testedComponent)).toBe(expectedOutput);

const fixture2 = fixtures.get(2);

await uiDialog.insertURL(gitRepo.fileUrl(fixture2.input().name, false));
```
