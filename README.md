# Opinionated rules to test writing (in TS but not only)

## Keep const together, and let together

Bad:

```ts
const environment = envConfig.environments.staging;
const possibleGitProviders = ['gitlab', 'github', 'bitbucket'];

let page: Login;
let spacePage: Space;

const CommonFn = new Common();
const testedComponent = 'myComponent';

let pageTitle: string;
let recordsWithData: Array<Record<string, string>>;
```

Good:

```ts
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

```ts
const pageTitle = CommonFn.pageTitle;

await createNewPage.activateComponent(testedComponent, 'click');

const fixture = fixtures.get(1);

await uiDialog.switchToTab('RemoteURL');
```

Good:
```ts
const pageTitle = CommonFn.pageTitle;
const fixture = fixtures.get(1);

await createNewPage.activateComponent(testedComponent, 'click');
await uiDialog.switchToTab('RemoteURL');
```

## If you need to prepare more data during test flow, group them together

Bad:
```ts
(...)
await uiDialog.switchToTab('RemoteURL');
expect(await confluencePage.getText(testedComponent)).toBe(fixture.output().content);

const gitRepo = new GitRepo(gitProvider, envConfig.gitServices);

await uiDialog.insertURL(gitRepo.fileUrl(tabsInputOutputFiles[2].fixture.input().name, false));

const selectedTab = faker.helpers.arrayElement(tabsInputOutputFiles);

await createNewPage.insertSaveComponent('insert', uiDialog, testedComponent);
```

Good:
```ts
(...)
await uiDialog.switchToTab('RemoteURL');
expect(await confluencePage.getText(testedComponent)).toBe(fixture.output().content);

const gitRepo = new GitRepo(gitProvider, envConfig.gitServices);
const selectedTab = faker.helpers.arrayElement(tabsInputOutputFiles);

await uiDialog.insertURL(gitRepo.fileUrl(tabsInputOutputFiles[2].fixture.input().name, false));
await createNewPage.insertSaveComponent('insert', uiDialog, testedComponent);

```



## Add line break after assertion

*For complex scenarios I use more than one expect in the test, not to lose time*

Bad:

```ts
await uiDialog.editURL(url);
expect(await confluencePage.getText(testedComponent)).toBe(expected);
await uiDialog.useAndCheckPreview(fixture.output().content, false, testedComponent);
await createNewPage.insertSaveComponent('save', uiDialog, testedComponent);
expect(await confluencePage.getText(testedComponent)).toBe(expected);
await createNewPage.publish();
```

Good:
```ts
await uiDialog.editURL(url);
expect(await confluencePage.getText(testedComponent)).toBe(expected);

await uiDialog.useAndCheckPreview(fixture.output().content, false, testedComponent);
await createNewPage.insertSaveComponent('save', uiDialog, testedComponent);
expect(await confluencePage.getText(testedComponent)).toBe(expected);

await createNewPage.publish();
```


## Add line break when switching between data preparation and step execution

Bad:
```ts
await confluencePage.editPage();
const uiDialog = "component";
const gitRepo = new GitRepo(secondGitProvider, envConfig.gitServices);
await uiDialog.editURL(gitRepo.fileUrl(fixtures.get(2).input().name, false));
```


Good:
```ts
await confluencePage.editPage();

const uiDialog = "component";
const gitRepo = new GitRepo(secondGitProvider, envConfig.gitServices);

await uiDialog.editURL(gitRepo.fileUrl(fixtures.get(2).input().name, false));
```

## Prepare new data after assertion

Sometimes it is not possible to prepare all data in the beginning of the test. In that case, prepare new data after assertion

Bad:
```ts
const fixture1 = fixtures.get(1);

await uiDialog.insertURL(gitRepo.fileUrl(fixture1.input().name, false));

const fixture2 = fixtures.get(2);

expect(await confluencePage.getText(testedComponent)).toBe(expectedOutput);

await uiDialog.insertURL(gitRepo.fileUrl(fixture2.input().name, false));
```


Good:
```ts
const fixture1 = fixtures.get(1);

await uiDialog.insertURL(gitRepo.fileUrl(fixture1.input().name, false));
expect(await confluencePage.getText(testedComponent)).toBe(expectedOutput);

const fixture2 = fixtures.get(2);

await uiDialog.insertURL(gitRepo.fileUrl(fixture2.input().name, false));
```
