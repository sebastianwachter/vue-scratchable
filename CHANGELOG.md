# Changelog

## v0.3.4

#### Bugfix:

- Fix CORS issues that used to happen when referencing external images.

#### Chore:

- Readme update fixing examples according to the vue's new linter rules and adding documentation regarding the usage of CORS with external images.

---

## v0.3.3

#### Chore:

- Update packages.

---

## v0.3.2

#### Bugfix:

- Fix the MutationObserver with the debounce script (this actually did never work in this state).

#### Cleanup:

- Replace `slotDomElement` with Vue's `$el`.

#### Chore: 

- Update packages (all packages with a dependency to `eslint` are currently breaking its package resolution in newer node and npm versions).

---

## v0.3.1

#### Chore:

- Readme update.

---

## v0.3.0

#### 🔥 BREAKING:

- Rename `percentageUpdate` event to `percentage-update`to comply with vue's new linter rules.

#### Chore:

- Update several packages.

---

## v0.2.3

#### Features:

- Initial release! 🎉🎉
