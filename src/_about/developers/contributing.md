---
layout: documentation
title: Contributing
permalink: /about/developers/contributing
has-parent: /about/developers/
intro-text: How to contribute code to the Design System.
anchors:
  - anchor: Modifying existing components
  - anchor: Writing CSS for the Design System
  - anchor: Contributing new components
---

The two main ways for developers to contribute to the Design System are by modifying existing components and contributing new components. Regardless of which type of contribution you are making, each PR should:

- have at least 90% test coverage
- have appropriate comments/documentation for functions, classes, etc.
- have Storybook stories for new features
- minimize complexity
- include only the smallest changeset required for the feature or fix

## Modifying existing components

PRs which make a change to the Design System should be manageable in size (less than ~500 lines of code). This is to meant to:

- Save your time as the developer
- Keep PRs tightly focused
- Keep the review process short.

## Writing CSS for the Design System

When naming components, be sure to use Formation’s [naming conventions](naming).

### Searchable selectors

Many of the features in Sass make it easy to use shorthand to reduce repetitive typing and write cleaner .scss files. However, this makes using search features in GitHub or text editors much more difficult because it is not always clear how the shorthand was written; finding the right query requires guesswork.

<div class="do-dont">
<div class="do-dont__do">
<h3 class="do-dont__heading">Do</h3>
<div class="do-dont__content" markdown="1">
Write out the full name of each selector.

```css
.alert {
}

.alert--warning {
}

.alert--error {
}
```
</div>
</div>
<div class="do-dont__dont">
<h3 class="do-dont__heading">Don’t</h3>
<div class="do-dont__content" markdown="1">
Don’t use Sass shorthand features, such as nesting with ampersands often used with BEM syntax.

```scss
.alert {
  &--warning {
  }
  &--error {
  }
}
```
</div>
</div>
</div>

## Contributing new components

This section details how to follow the experimental design process to contribute new components. If you haven't read it already, refer to the [contributing to the design system]({{ site.baseurl }}/about/contributing-to-the-design-system) page for more information about the full process.

### Writing experimental design code

Each experimental design should:
- Be absent of business logic and domain knowledge
- Not import application code
- Not introduce breaking changes

Developing the experiment as if it were a standalone library will make the code more reusable and graduating the component or pattern into the official design system smoother.

Each experimental design should [include a README](#readme) and be [owned by a team](#codeowners).

### Sharing experimental design code

Sharing code between applications is necessarily more involved than writing code for a single application. To avoid the overhead that sharing code introduces (reporting development status, managing breaking changes, deprecating the code etc.), it's recommended to develop experimental designs in the application directory and not import it from another application.

If the time comes that another application needs to use the experiment, the rest of this section describes the process for how to share this code.

### Code location

Each experimental design is located in its own directory in [`vets-website`](https://github.com/department-of-veterans-affairs/vets-website/) at `src/experimental/` unless otherwise noted in its documentation on this site.

**Example:**
If your team needs an experimental button that's larger than the standard button, you would create `src/experimental/large-button/index.jsx` as the entry file for your "library."

### README

Each experimental design should have a README that contains the following information:
- Development status: `stable`, `unstable`, or `deprecated`
     - The `unstable` status means the code is under active development and the public API may change without notice
     - The `stable` status means the public API is finalized, but the code may still receive backward-compatible updates such as accessibility improvements and bug fixes
     - The `deprecated` status means the code should no longer be used in applications
          - This may be because of a breaking change (see [Breaking changes](#breaking-changes) below), official adoption into the design system, or research which indicates the experiment was unsuccessful
          - See [Ending the experiment](#ending-the-experiment) below for instructions on what to do when deprecating code
- API documentation (optional but encouraged)

### Breaking changes
"Breaking changes" is defined here In `semver` terms as a backwards incompatible change to the public API of your component or pattern. (See the [Semantic Versioning Specification](https://semver.org/#spec-item-8) for more details.)

Once the code for an experimental design is stable, **breaking changes should not be introduced.** Other applications may depend on this code, but are unable to pin the version because it's not a "proper" library.

If you need to introduce breaking changes, **do not modify the existing code.** Instead, copy the contents of the directory to a sibling directory post-fixed with a version number.

**Example:**
The `LargeButton` you created accepted `children`, but because of reasons, you need to limit the content of the button to only text. You've decided to remove the `children` prop and add a `label` prop instead which accepts only strings. To introduce this change, you would:
1. Copy the contents of `src/experimental/large-button/` to `src/experimental/large-button-2`
2. Update the status in `src/experimental/large-button/README` to `deprecated` and indicate why (because there's a new version)
3. Make the breaking changes to `src/experimental/large-button-2`
4. Change the import statements `'experimental/large-button-2'` in your application
5. Update the [CODEOWNERS file](#codeowners) to add the new directory
6. Make an announcement for anybody who may be using the deprecated code

### CODEOWNERS
Add your team's GitHub team name to the [CODEOWNERS file](https://github.com/department-of-veterans-affairs/vets-website/blob/master/.github/CODEOWNERS) to take ownership of the experiment's code. This will mean your team will be required reviewers on all changes to this code.

### Test coverage
As with all code, test coverage is critical. This is especially true with shared code. Aim for at least 90% unit test coverage before declaring an experiment to be `stable`.

### Using shared experimental designs

Before using an experimental design, first check the `src/experimental/` directory in `vets-website` to see if it's been shared yet. If not, work with the authoring team to move the code into `src/experimental/`. See [Sharing experimental design code](#sharing-experimental-design-code) above for more information.

The babel module resolver plugin has the `root` set to `"./src"`, so you can import your experimental design with the following:

 ```js
import LargeButton from '~/experimental/large-button';
```

### Ending the experiment
Experimental designs are meant to be short-lived. The experimental design code may no longer be needed because:
- The design was approved for adoption into the design system
- The design was rejected for adoption into the design system
- A breaking change was introduced and a new version was created

When code is deprecated for any of these reasons, the goal is to delete the code. If there are no applications using the deprecated code, simply delete the directory.

If there are applications using the deprecated code:
1. In the README: Mark the code as `deprecated`
1. In the README: Clearly outline what engineers should do to stop using the experiment
    - This may be something like "upgrade to `~/experimental/large-button-2`," "use `@department-of-veterans-affairs/component-library/LargeButton`," or "discontinue use; the experiment has been rejected"
1. In Slack: Notify teams that the code has been deprecated, either via an announcement or reaching out directly to the teams using the experiment
1. Check in weekly to see if there are still applications using the experiment; delete the directory when no applications are dependent on it

It's the responsibility of the code owners to delete deprecated code when it's no longer in use.