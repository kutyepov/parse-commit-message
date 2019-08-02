# parse-commit-message [![npm version][npmv-img]][npmv-url] [![github release][ghrelease-img]][ghrelease-url] [![License][license-img]][license-url]

> Extensible parser for git commit messages following Conventional Commits Specification

Please consider following this project's author, [Charlike Mike Reagent](https://github.com/tunnckoCore), and :star: the project to show your :heart: and support.

<div id="thetop"></div>

[![Code style][codestyle-img]][codestyle-url]
[![CircleCI linux build][linuxbuild-img]][linuxbuild-url]
[![CodeCov coverage status][codecoverage-img]][codecoverage-url]
[![DavidDM dependency status][dependencies-img]][dependencies-url]
[![Renovate App Status][renovateapp-img]][renovateapp-url]
[![Make A Pull Request][prs-welcome-img]][prs-welcome-url]
[![Semantically Released][new-release-img]][new-release-url]

If you have any _how-to_ kind of questions, please read the [Contributing Guide](./CONTRIBUTING.md) and [Code of Conduct](./CODE_OF_CONDUCT.md) documents.  
For bugs reports and feature requests, [please create an issue][open-issue-url] or ping
[@tunnckoCore](https://twitter.com/tunnckoCore) at Twitter.

[![Become a Patron][patreon-img]][patreon-url]
[![Conventional Commits][ccommits-img]][ccommits-url]
[![NPM Downloads Weekly][downloads-weekly-img]][npmv-url]
[![NPM Downloads Monthly][downloads-monthly-img]][npmv-url]
[![NPM Downloads Total][downloads-total-img]][npmv-url]
[![Share Love Tweet][shareb]][shareu]

Project is [semantically](https://semver.org) & automatically published with [new-release](https://github.com/tunnckoCore/release-cli) on [CircleCI](https://circleci.com) and released by [New Release](https://github.com/apps/tunnckocore-release) GitHub App.

<!-- Logo when needed:

<p align="center">
  <a href="https://github.com/tunnckoCoreLabs/parse-commit-message">
    <img src="./media/logo.png" width="85%">
  </a>
</p>

-->

## Highlights

- Understands and follows [Conventional Commits](https://conventionalcommits.org/) Specification
- Closed cycle: utilities for parsing, stringifying **and** validation
- Infinitely extensible through plugins, [two built-in](#mappers)
- Thoroughly tested with 100% coverage and 64+ tests
- Collecting mentions from commit message
- Detection of breaking changes in commit

## Table of Contents

- [Install](#install)
- [Type definitions](#type-definitions)
  * [Header](#header)
  * [Commit](#commit)
  * [Mention](#mention)
- [API](#api)
  * [src/commit.js](#srccommitjs)
    + [.parseCommit](#parsecommit)
    + [.stringifyCommit](#stringifycommit)
    + [.validateCommit](#validatecommit)
    + [.checkCommit](#checkcommit)
  * [src/header.js](#srcheaderjs)
    + [.parseHeader](#parseheader)
    + [.stringifyHeader](#stringifyheader)
    + [.validateHeader](#validateheader)
    + [.checkHeader](#checkheader)
  * [src/index.js](#srcindexjs)
    + [.applyPlugins](#applyplugins)
    + [.plugins](#plugins)
    + [.mappers](#mappers)
  * [src/main.js](#srcmainjs)
    + [.parse](#parse)
    + [.stringify](#stringify)
    + [.validate](#validate)
    + [.check](#check)
  * [src/plugins/increment.js](#srcpluginsincrementjs)
    + [increment](#increment)
  * [src/plugins/mentions.js](#srcpluginsmentionsjs)
    + [mentions](#mentions)
- [See Also](#see-also)
- [Contributing](#contributing)
  * [Follow the Guidelines](#follow-the-guidelines)
  * [Support the project](#support-the-project)
  * [OPEN Open Source](#open-open-source)
  * [Wonderful Contributors](#wonderful-contributors)
- [License](#license)

_(TOC generated by [verb](https://github.com/verbose/verb) using [markdown-toc](https://github.com/jonschlinkert/markdown-toc))_

## Install

This project requires [**Node.js**](https://nodejs.org) **^8.10.0 || >=10.13.0**. Install it using
[**yarn**](https://yarnpkg.com) or [**npm**](https://npmjs.com).  
_We highly recommend to use Yarn when you think to contribute to this project._

```bash
$ yarn add parse-commit-message
```

## Type definitions

For TypeScript support, please consider sending a PR here (adding `src/index.d.ts`)
or inform us when you PR to the DefinitelyTyped.

For FlowType support, PR adding `.js.flow` files in the `src/` for every respective file.

### Header

```ts
type Header = {
  type: string;
  scope?: string | null;
  subject: string;
};
```

The `scope` may exist or not. When exist it should be non-empty string.

### Commit

```ts
type Commit = {
  header: Header;
  body?: string | null;
  footer?: string | null;
  increment?: string | boolean;
  isBreaking?: boolean;
  mentions?: Array<Mention>;
};
```

Note: It may also include properties set by the plugins - the `isBreaking`, `increment` and `mentions` 
are such. They are there when you apply the `increment` and `mentions` plugins.

ProTip: The `increment` may be a Boolean `false` if the commit type, for example, is `chore`. 

See [.applyPlugins](#applyplugins) and [.plugins](#plugins).

### Mention

```ts
type Mention = {
  handle: string;
  mention: string;
  index: number;
};
```

See [collect-mentions][] for more.

## API

<!-- docks-start -->
_Generated using [docks](http://npm.im/docks)._

### [src/commit.js](/src/commit.js)

#### [.parseCommit](/src/commit.js#L31)
Receives a full commit message `string` and parses it into an `Commit` object
and returns it.
Basically the same as [.parse](#parse), except that
it only can accept single string.

_The `parse*` methods are not doing any checking and validation,
so you may want to pass the result to `validateCommit` or `checkCommit`,
or to `validateCommit` with `ret` option set to `true`._

**Params**
- `commit` **{string}** a message like `'fix(foo): bar baz\n\nSome awesome body!'`

**Returns**
- `Commit` a standard object like `{ header: Header, body?, footer? }`

**Examples**
```javascript
import { parseCommit } from 'parse-commit-message';

const commitObj = parseCommit('foo: bar qux\n\nokey dude');
console.log(commitObj);
// => {
//   header: { type: 'foo', scope: null, subject: 'bar qux' },
//   body: 'okey dude',
//   footer: null,
// }
```

#### [.stringifyCommit](/src/commit.js#L62)
Receives a `Commit` object, validates it using `validateCommit`,
builds a "commit" string and returns it. Method throws if problems found.
Basically the same as [.stringify](#stringify), except that
it only can accept single `Commit` object.

**Params**
- `commit` **{Commit}** a `Commit` object like `{ header: Header, body?, footer? }`

**Returns**
- `string` a commit nessage stirng like `'fix(foo): bar baz'`

**Examples**
```javascript
import { stringifyCommit } from 'parse-commit-message';

const commitStr = stringifyCommit({
  header: { type: 'foo', subject: 'bar qux' },
  body: 'okey dude',
});
console.log(commitStr); // => 'foo: bar qux\n\nokey dude'
```

#### [.validateCommit](/src/commit.js#L110)
Validates given `Commit` object and returns `boolean`.
You may want to set `ret` to `true` return an object instead of throwing.
Basically the same as [.validate](#validate), except that
it only can accept single `Commit` object.

**Params**
- `commit` **{Commit}** a `Commit` like `{ header: Header, body?, footer? }`
- `[ret]` **{boolean}** to return result instead of throwing, default `false`

**Returns**
- `undefined` if `ret` is `true` then returns `{ value, error }` object,
                         where `value` is `Commit` and `error` a standard `Error`

**Examples**
```javascript
import { validateCommit } from 'parse-commit-message';

const commit = {
  header: { type: 'foo', subject: 'bar qux' },
  body: 'okey dude',
};

const commitIsValid = validateCommit(commit);
console.log(commitIsValid); // => true

const { value } = validateCommit(commit, true);
console.log(value);
// => {
//   header: { type: 'foo', scope: null, subject: 'bar qux' },
//   body: 'okey dude',
//   footer: null,
// }
```

#### [.checkCommit](/src/commit.js#L139)
Receives a `Commit` and checks if it is valid. Method throws if problems found.
Basically the same as [.check](#check), except that
it only can accept single `Commit` object.

**Params**
- `commit` **{Commit}** a `Commit` like `{ header: Header, body?, footer? }`

**Returns**
- `Commit` returns the same as given if no problems, otherwise it will throw.

**Examples**
```javascript
import { checkCommit } from 'parse-commit-message';

try {
  checkCommit({ header: { type: 'fix' } });
} catch(err) {
  console.log(err);
  // => TypeError: header.subject should be non empty string
}

// throws because can accept only Commit objects
checkCommit('foo bar baz');
checkCommit(123);
checkCommit([{ header: { type: 'foo', subject: 'bar' } }]);
```

### [src/header.js](/src/header.js)

#### [.parseHeader](/src/header.js#L29)
Parses given `header` string into an header object.
Basically the same as [.parse](#parse), except that
it only can accept single string and returns a `Header` object.

_The `parse*` methods are not doing any checking and validation,
so you may want to pass the result to `validateHeader` or `checkHeader`,
or to `validateHeader` with `ret` option set to `true`._

**Params**
- `header` **{string}** a header stirng like `'fix(foo): bar baz'`

**Returns**
- `Header` a `Header` object like `{ type, scope?, subject }`

**Examples**
```javascript
import { parseHeader } from 'parse-commit-message';

const longCommitMsg = `fix: bar qux

Awesome body!`;

const headerObj = parseCommit(longCommitMsg);
console.log(headerObj);
// => { type: 'fix', scope: null, subject: 'bar qux' }
```

#### [.stringifyHeader](/src/header.js#L66)
Receives a `header` object, validates it using `validateHeader`,
builds a "header" string and returns it. Method throws if problems found.
Basically the same as [.stringify](#stringify), except that
it only can accept single `Header` object.

**Params**
- `header` **{Header}** a `Header` object like `{ type, scope?, subject }`

**Returns**
- `string` a header stirng like `'fix(foo): bar baz'`

**Examples**
```javascript
import { stringifyHeader } from 'parse-commit-message';

const headerStr = stringifyCommit({ type: 'foo', subject: 'bar qux' });
console.log(headerStr); // => 'foo: bar qux'
```

#### [.validateHeader](/src/header.js#L114)
Validates given `header` object and returns `boolean`.
You may want to pass `ret` to return an object instead of throwing.
Basically the same as [.validate](#validate), except that
it only can accept single `Header` object.

**Params**
- `header` **{Header}** a `Header` object like `{ type, scope?, subject }`
- `[ret]` **{boolean}** to return result instead of throwing, default `false`

**Returns**
- `undefined` if `ret` is `true` then returns `{ value, error }` object,
                         where `value` is `Header` and `error` a standard `Error`

**Examples**
```javascript
import { validateHeader } from 'parse-commit-message';

const header = { type: 'foo', subject: 'bar qux' };

const headerIsValid = validateHeader(header);
console.log(headerIsValid); // => true

const { value } = validateHeader(header, true);
console.log(value);
// => {
//   header: { type: 'foo', scope: null, subject: 'bar qux' },
//   body: 'okey dude',
//   footer: null,
// }

const { error } = validateHeader({
  type: 'bar'
}, true);

console.log(error);
// => TypeError: header.subject should be non empty string
```

#### [.checkHeader](/src/header.js#L144)
Receives a `Header` and checks if it is valid.
Basically the same as [.check](#check), except that
it only can accept single `Header` object.

**Params**
- `header` **{Header}** a `Header` object like `{ type, scope?, subject }`

**Returns**
- `Header` returns the same as given if no problems, otherwise it will throw.

**Examples**
```javascript
import { checkHeader } from 'parse-commit-message';

try {
  checkHeader({ type: 'fix' });
} catch(err) {
  console.log(err);
  // => TypeError: header.subject should be non empty string
}

// throws because can accept only Header objects
checkHeader('foo bar baz');
checkHeader(123);
checkHeader([]);
checkHeader([{ type: 'foo', subject: 'bar' }]);
```

### [src/index.js](/src/index.js)

#### [.applyPlugins](/src/index.js#L87)
Apply a set of `plugins` over all of the given `commits`.
A plugin is a simple function passed with `Commit` object,
which may be returned to modify and set additional properties
to the `Commit` object.

_The `commits` should be coming from `parse`, `validate` (with `ret` option)
or the `check` methods. It does not do checking and validation._

**Params**
- `plugins` **{Array&lt;Function&gt;}** a simple function like `(commit) => {}`
- `commits` **{string|object|array}** a value which should already be gone through `parse`

**Returns**
- `Array<Commit>` plus the modified or added properties from each function in `plugins`

**Examples**
```javascript
import dedent from 'dedent';
import { applyPlugins, plugins, parse, check } from './src';

const commits = [
  'fix: bar qux',
  dedent`feat(foo): yea yea

  Awesome body here with @some mentions
  resolves #123

  BREAKING CHANGE: ouch!`,
  'chore(ci): updates for ci config',
  {
    header: { type: 'fix', subject: 'Barry White' },
    body: 'okey dude',
    foo: 'possible',
  },
];

// Parses, normalizes, validates
// and applies plugins
const results = applyPlugins(plugins, check(parse(commits)));

console.log(results);
// => [ { body: null,
//   footer: null,
//   header: { scope: null, type: 'fix', subject: 'bar qux' },
//   mentions: [],
//   increment: 'patch',
//   isBreaking: false },
// { body: 'Awesome body here with @some mentions\nresolves #123',
//   footer: 'BREAKING CHANGE: ouch!',
//   header: { scope: 'foo', type: 'feat', subject: 'yea yea' },
//   mentions: [ [Object] ],
//   increment: 'major',
//   isBreaking: true },
// { body: null,
//   footer: null,
//   header:
//    { scope: 'ci', type: 'chore', subject: 'updates for ci config' },
//   mentions: [],
//   increment: false,
//   isBreaking: false },
// { body: 'okey dude',
//   footer: null,
//   header: { scope: null, type: 'fix', subject: 'Barry White' },
//   foo: 'possible',
//   mentions: [],
//   increment: 'patch',
//   isBreaking: false } ]
```

#### [.plugins](/src/index.js#L155)
An array which includes `mentions` and `increment` built-in plugins.
The `mentions` is an array of objects. Basically what's returned from
the [collect-mentions][] package.

**Examples**
```javascript
import { plugins, applyPlugins, parse } from 'parse-commit-message';

console.log(plugins); // =>  [mentions, increment]
console.log(plugins[0]); // => [Function mentions]
console.log(plugins[1]); // => [Function increment]

const cmts = parse([
  'fix: foo @bar @qux haha',
  'feat(cli): awesome @tunnckoCore feature\n\nSuper duper baz!',
  'fix: ooh\n\nBREAKING CHANGE: some awful api change'
]);

const commits = applyPlugins(plugins, cmts);
console.log(commits);
// => [
//   {
//     header: { type: 'fix', scope: '', subject: 'foo bar baz' },
//     body: '',
//     footer: '',
//     increment: 'patch',
//     isBreaking: false,
//     mentions: [
//       { handle: '@bar', mention: 'bar', index: 8 },
//       { handle: '@qux', mention: 'qux', index: 13 },
//     ]
//   },
//   {
//     header: { type: 'feat', scope: 'cli', subject: 'awesome feature' },
//     body: 'Super duper baz!',
//     footer: '',
//     increment: 'minor',
//     isBreaking: false,
//     mentions: [
//       { handle: '@tunnckoCore', mention: 'tunnckoCore', index: 18 },
//     ]
//   },
//   {
//     header: { type: 'fix', scope: '', subject: 'ooh' },
//     body: 'BREAKING CHANGE: some awful api change',
//     footer: '',
//     increment: 'major',
//     isBreaking: true,
//     mentions: [],
//   },
// ]
```

#### [.mappers](/src/index.js#L188)
An object (named set) which includes `mentions` and `increment` built-in plugins.

**Examples**
```javascript
import { mappers, applyPlugins, parse } from 'parse-commit-message';

console.log(mappers); // => { mentions, increment }
console.log(mappers.mentions); // => [Function mentions]
console.log(mappers.increment); // => [Function increment]

const flat = true;
const parsed = parse('fix: bar', flat);
console.log(parsed);
// => {
//   header: { type: 'feat', scope: 'cli', subject: 'awesome feature' },
//   body: 'Super duper baz!',
//   footer: '',
// }

const commit = applyPlugins([mappers.increment], parsed);
console.log(commit)
// => [{
//   header: { type: 'feat', scope: 'cli', subject: 'awesome feature' },
//   body: 'Super duper baz!',
//   footer: '',
//   increment: 'patch',
// }]
```

### [src/main.js](/src/main.js)

#### [.parse](/src/main.js#L51)
Receives and parses a single or multiple commit message(s) in form of string,
object, array of strings, array of objects or mixed.

**Params**
- `commits` **{string|object|array}** a value to be parsed into an object like `Commit` type
- `[flat]` **{boolean}** if the returned result length is 1, then returns the first item

**Returns**
- `Array<Commit>` if `flat: true`, returns a `Commit`

**Examples**
```javascript
import { parse } from 'parse-commit-message';

const commits = [
  'fix(ci): tweaks for @circleci config',
  'chore: bar qux'
];
const result = parse(commits);
console.log(result);
// => [{
//   header: { type: 'fix', scope: 'ci', subject: 'tweaks for @circleci config' },
//   body: null,
//   footer: null,
// }, {
//   header: { type: 'chore', scope: null, subject: 'bar qux' },
//   body: null,
//   footer: null,
// }]

// Or pass `flat = true` to return a single object
// when results contain only one item.
const commitMessage = `feat: awesome yeah

Awesome body!
resolves #123

Signed-off-by: And Footer <abc@exam.pl>`;

const res = parse(commitMessage, true);

console.log(res);
// => {
//   header: { type: 'feat', scope: null, subject: 'awesome yeah' },
//   body: 'Awesome body!\nresolves #123',
//   footer: 'Signed-off-by: And Footer <abc@exam.pl>',
// }
```

#### [.stringify](/src/main.js#L101)
Receives a `Commit` object, validates it using `validate`,
builds a "commit" message string and returns it.

This method does checking and validation too,
so if you pass a string, it will be parsed and validated,
and after that turned again to string.

**Params**
- `commits` **{string|object|array}** a `Commit` object, or anything that can be passed to `check`
- `[flat]` **{boolean}** if the returned result length is 1, then returns the first item

**Returns**
- `Array<string>` an array of commit strings like `'fix(foo): bar baz'`,
                    if `flat: true`, returns a `string`

**Examples**
```javascript
import { parse, stringify } from 'parse-commit-message';

const commitMessage = `feat: awesome yeah

Awesome body!
resolves #123

Signed-off-by: And Footer <abc@exam.pl>`;

const flat = true;
const res = parse(commitMessage, flat);

const str = stringify(res, flat);
console.log(str);
console.log(str === commitMessage);
```

#### [.validate](/src/main.js#L174)
Validates a single or multiple commit message(s) in form of string,
object, array of strings, array of objects or mixed.
You may want to pass `ret` to return an object instead of throwing.

**Params**
- `commits` **{string|object|array}** a value to be parsed & validated into an object like `Commit` type
- `[ret]` **{boolean}** to return result instead of throwing, default `false`

**Returns**
- `undefined` if `ret` is `true` then returns `{ value, error }` object,
                         where `value` is `Commit|Array<Commit>` and `error` a standard `Error`

**Examples**
```javascript
import { validate } from 'parse-commit-message';

console.log(validate('foo bar qux')); // false
console.log(validate('foo: bar qux')); // true
console.log(validate('fix(ci): bar qux')); // true

console.log(validate(['a bc cqux', 'foo bar qux'])); // false

console.log(validate({ qux: 1 })); // false
console.log(validate({ header: { type: 'fix' } })); // false
console.log(validate({ header: { type: 'fix', subject: 'ok' } })); // true

const commitObject = {
  header: { type: 'test', subject: 'updating tests' },
  foo: 'bar',
  isBreaking: false,
  body: 'oh ah',
};
console.log(validate(commitObject)); // true

const result = validate('foo bar qux', true);
console.log(result.error);
// => Error: expect \`commit\` to follow:
// <type>[optional scope]: <description>
//
// [optional body]
//
// [optional footer]

const res = validate('fix(ci): okey barry', true);
console.log(result.value);
// => [{
//   header: { type: 'fix', scope: 'ci', subject: 'okey barry' },
//   body: null,
//   footer: null,
// }]

const commit = { header: { type: 'fix' } };
const { error } = validate(commit, true);
console.log(error);
// => TypeError: header.subject should be non empty string

const commit = { header: { type: 'fix', scope: 123, subject: 'okk' } };
const { error } = validate(commit, true);
console.log(error);
// => TypeError: header.scope should be non empty string when given
```

#### [.check](/src/main.js#L211)
Receives a single or multiple commit message(s) in form of string,
object, array of strings, array of objects or mixed.

Basically the return result is the same as if you run `.validate()` with
the `ret` option, but instead it throws if find problems.

**Params**
- `commits` **{string|object|array}** a value to be parsed & validated into an object like `Commit` type
- `[flat]` **{boolean}** if the returned result length is 1, then returns the first item

**Returns**
- `Array<Commit>` returns the same as given if no problems, otherwise it will throw;
                    if `flat: true`, returns a `Commit`

**Examples**
```javascript
import { check } from 'parse-commit-message';

try {
  check({ header: { type: 'fix' } });
} catch(err) {
  console.log(err);
  // => TypeError: header.subject should be non empty string
}

// Can also validate/check a strings, array of strings,
// or even mixed - array of strings and objects
try {
  check('fix(): invalid scope, it cannot be empty')
} catch(err) {
  console.log(err);
  // => TypeError: header.scope should be non empty string when given
}
```

### [src/plugins/increment.js](/src/plugins/increment.js)

#### [increment](/src/plugins/increment.js#L13)
A plugin that adds `increment` and `isBreaking` properties
to the `commit`. It is already included in the `plugins` named export,
and in `mappers` named export.

_See the [.plugins](#plugins) and [.mappers](#mappers)  examples._

**Params**
- `commit` **{Commit}** a standard `Commit` object

**Returns**
- `Commit` plus `{ increment: string, isBreaking: boolean }`

### [src/plugins/mentions.js](/src/plugins/mentions.js)

#### [mentions](/src/plugins/mentions.js#L17)
A plugin that adds `mentions` array property to the `commit`.
It is already included in the `plugins` named export,
and in `mappers` named export.
Basically each entry in that array is an object,
directly returned from the [collect-mentions][].

_See the [.plugins](#plugins) and [.mappers](#mappers)  examples._

**Params**
- `commit` **{Commit}** a standard `Commit` object

**Returns**
- `Commit` plus `{ mentions: Array<Mention> }`

<!-- docks-end -->

**[back to top](#thetop)**

## See Also

Some of these projects are used here or were inspiration for this one, others are just related. So, thanks for your
existance!
- [@tunnckocore/config](https://www.npmjs.com/package/@tunnckocore/config): All the configs for all the tools, in one place | [homepage](https://github.com/tunnckoCoreLabs/config "All the configs for all the tools, in one place")
- [@tunnckocore/create-project](https://www.npmjs.com/package/@tunnckocore/create-project): Create and scaffold a new project, its GitHub repository and contents | [homepage](https://github.com/tunnckoCoreLabs/create-project "Create and scaffold a new project, its GitHub repository and contents")
- [@tunnckocore/execa](https://www.npmjs.com/package/@tunnckocore/execa): Thin layer on top of [execa][] that allows executing multiple commands… [more](https://github.com/tunnckoCoreLabs/execa) | [homepage](https://github.com/tunnckoCoreLabs/execa "Thin layer on top of [execa][] that allows executing multiple commands in parallel or in sequence")
- [@tunnckocore/scripts](https://www.npmjs.com/package/@tunnckocore/scripts): Universal and minimalist scripts & tasks runner. | [homepage](https://github.com/tunnckoCoreLabs/scripts "Universal and minimalist scripts & tasks runner.")
- [@tunnckocore/update](https://www.npmjs.com/package/@tunnckocore/update): Update to latest project files and templates, based on `charlike` scaffolder | [homepage](https://github.com/tunnckoCoreLabs/update "Update to latest project files and templates, based on `charlike` scaffolder")
- [asia](https://www.npmjs.com/package/asia): Blazingly fast, magical and minimalist testing framework, for Today and Tomorrow | [homepage](https://github.com/olstenlarck/asia#readme "Blazingly fast, magical and minimalist testing framework, for Today and Tomorrow")
- [charlike](https://www.npmjs.com/package/charlike): Small, fast and streaming project scaffolder with support for hundreds of… [more](https://github.com/tunnckoCoreLabs/charlike) | [homepage](https://github.com/tunnckoCoreLabs/charlike "Small, fast and streaming project scaffolder with support for hundreds of template engines and sane defaults")
- [docks](https://www.npmjs.com/package/docks): Extensible system for parsing and generating documentation. It just freaking works! | [homepage](https://github.com/tunnckoCore/docks "Extensible system for parsing and generating documentation. It just freaking works!")
- [git-commits-since](https://www.npmjs.com/package/git-commits-since): Get all commits since given period of time or by default… [more](https://github.com/tunnckoCoreLabs/git-commits-since) | [homepage](https://github.com/tunnckoCoreLabs/git-commits-since "Get all commits since given period of time or by default from latest git semver tag. Understands and follows both SemVer and the Conventional Commits specification.")
- [gitcommit](https://www.npmjs.com/package/gitcommit): Lightweight and joyful `git commit` replacement. Conventional Commits compliant. | [homepage](https://github.com/tunnckoCore/gitcommit "Lightweight and joyful `git commit` replacement. Conventional Commits compliant.")

**[back to top](#thetop)**

## Contributing

### Follow the Guidelines

Please read the [Contributing Guide](./CONTRIBUTING.md) and [Code of Conduct](./CODE_OF_CONDUCT.md) documents for advices.  
For bugs reports and feature requests, [please create an issue][open-issue-url] or ping
[@tunnckoCore](https://twitter.com/tunnckoCore) at Twitter.

### Support the project

[Become a Partner or Sponsor?][patreon-url] :dollar: Check the **Partner**, **Sponsor** or **Omega-level** tiers! :tada: You can get your company logo, link & name on this file. It's also rendered on package page in [npmjs.com][npmv-url] and [yarnpkg.com](https://yarnpkg.com/en/package/parse-commit-message) sites too! :rocket:

Not financial support? Okey! [Pull requests](https://github.com/tunnckoCoreLabs/contributing#opening-a-pull-request), stars and all kind of [contributions](https://opensource.guide/how-to-contribute/#what-it-means-to-contribute) are always
welcome. :sparkles:

### OPEN Open Source

This project is following [OPEN Open Source](http://openopensource.org) model

> Individuals making significant and valuable contributions are given commit-access to the project to contribute as they see fit. This project is built on collective efforts and it's not strongly guarded by its founders.

There are a few basic ground-rules for its contributors

1. Any **significant modifications** must be subject to a pull request to get feedback from other contributors.
2. [Pull requests](https://github.com/tunnckoCoreLabs/contributing#opening-a-pull-request) to get feedback are _encouraged_ for any other trivial contributions, but are not required.
3. Contributors should attempt to adhere to the prevailing code-style and development workflow.

### Wonderful Contributors

Thanks to the hard work of these wonderful people this project is alive! It follows the
[all-contributors](https://github.com/kentcdodds/all-contributors) specification.  
Don't hesitate to add yourself to that list if you have made any contribution! ;) [See how,
here](https://github.com/jfmengels/all-contributors-cli#usage).

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore -->
| [<img src="https://avatars3.githubusercontent.com/u/5038030?v=4" width="120px;"/><br /><sub><b>Charlike Mike Reagent</b></sub>](https://tunnckocore.com)<br />[💻](https://github.com/tunnckoCoreLabs/parse-commit-message/commits?author=tunnckoCore "Code") [📖](https://github.com/tunnckoCoreLabs/parse-commit-message/commits?author=tunnckoCore "Documentation") [💬](#question-tunnckoCore "Answering Questions") [👀](#review-tunnckoCore "Reviewed Pull Requests") [🔍](#fundingFinding-tunnckoCore "Funding Finding") |
| :---: |

<!-- ALL-CONTRIBUTORS-LIST:END -->

Consider showing your [support](#support-the-project) to them. :sparkling_heart:

## License

Copyright (c) 2017-present, [Charlike Mike Reagent](https://tunnckocore.com) `<mameto2011@gmail.com>` & [contributors](#wonderful-contributors).  
Released under the [Apache-2.0 License][license-url].

<!-- Heading badges -->

[npmv-url]: https://www.npmjs.com/package/parse-commit-message
[npmv-img]: https://badgen.net/npm/v/parse-commit-message?icon=npm

[ghrelease-url]: https://github.com/tunnckoCoreLabs/parse-commit-message/releases/latest
[ghrelease-img]: https://badgen.net/github/release/tunnckoCoreLabs/parse-commit-message?icon=github

[license-url]: https://github.com/tunnckoCoreLabs/parse-commit-message/blob/master/LICENSE
[license-img]: https://badgen.net/npm/license/parse-commit-message

<!-- Front line badges -->

[codestyle-url]: https://github.com/airbnb/javascript
[codestyle-img]: https://badgen.net/badge/code%20style/airbnb/ff5a5f?icon=airbnb

[linuxbuild-url]: https://circleci.com/gh/tunnckoCoreLabs/parse-commit-message/tree/master
[linuxbuild-img]: https://badgen.net/circleci/github/tunnckoCoreLabs/parse-commit-message/master?label=build&icon=circleci

[codecoverage-url]: https://codecov.io/gh/tunnckoCoreLabs/parse-commit-message
[codecoverage-img]: https://badgen.net/codecov/c/github/tunnckoCoreLabs/parse-commit-message?icon=codecov

[dependencies-url]: https://david-dm.org/tunnckoCoreLabs/parse-commit-message
[dependencies-img]: https://badgen.net/david/dep/tunnckoCoreLabs/parse-commit-message?label=deps

[ccommits-url]: https://conventionalcommits.org/
[ccommits-img]: https://badgen.net/badge/conventional%20commits/v1.0.0/dfb317
[new-release-url]: https://ghub.io/new-release
[new-release-img]: https://badgen.net/badge/semantically/released/05c5ff

[downloads-weekly-img]: https://badgen.net/npm/dw/parse-commit-message
[downloads-monthly-img]: https://badgen.net/npm/dm/parse-commit-message
[downloads-total-img]: https://badgen.net/npm/dt/parse-commit-message

[renovateapp-url]: https://renovatebot.com
[renovateapp-img]: https://badgen.net/badge/renovate/enabled/green
[prs-welcome-img]: https://badgen.net/badge/PRs/welcome/green
[prs-welcome-url]: http://makeapullrequest.com
[paypal-donate-url]: https://paypal.me/tunnckoCore/10
[paypal-donate-img]: https://badgen.net/badge/$/support/purple
[patreon-url]: https://www.patreon.com/bePatron?u=5579781
[patreon-img]: https://badgen.net/badge/patreon/tunnckoCore/F96854?icon=patreon
[patreon-sponsor-img]: https://badgen.net/badge/become/a%20sponsor/F96854?icon=patreon

[shareu]: https://twitter.com/intent/tweet?text=https://github.com/tunnckoCoreLabs/parse-commit-message&via=tunnckoCore
[shareb]: https://badgen.net/badge/twitter/share/1da1f2?icon=twitter
[open-issue-url]: https://github.com/tunnckoCoreLabs/parse-commit-message/issues/new

[collect-mentions]: https://github.com/olstenlarck/collect-mentions
[detect-next-version]: https://github.com/tunnckoCoreLabs/detect-next-version
[execa]: https://github.com/sindresorhus/execa
[new-release]: https://github.com/tunnckoCore/new-release
[parse-commit-message]: https://github.com/tunnckoCoreLabs/parse-commit-message
[recommended-bump]: https://github.com/tunnckoCoreLabs/recommended-bump
[semantic-release]: https://github.com/semantic-release/semantic-release
