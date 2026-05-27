# Contribution Guidelines

## Contribution Rules for Each Repository

This project involves multiple code repositories. Since these repositories have different origins and serve different business purposes, their contribution rules also vary.

<table>
  <thead>
    <tr>
      <th>Repository</th>
      <th>Content</th>
      <th>Source</th>
      <th>Purpose</th>
      <th>Contribution Guidelines</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://atomgit.com/Harmonybrew/brew">brew</a></td>
      <td>Homebrew Client</td>
      <td>Forked from upstream</td>
      <td>The HarmonyOS version of the Homebrew package manager downloaded by users comes from here. </td>
      <td>
          1. Use English commit messages and comments. Commit message rules must align with those of the upstream repository. <br>
          2. Minimize modifications to the upstream code as much as possible. Do not submit changes based on personal preferences or custom requirements for your own company, and do not attempt to “optimize” the upstream code.
      </td>
    </tr>
    <tr>
      <td><a href="https://atomgit.com/Harmonybrew/install">install</a></td>
      <td>Homebrew installation script</td>
      <td>Forked from upstream</td>
      <td>The installation script used by users on HarmonyOS comes from here.</td>
      <td>
          1. Use English for commit messages and comments. Commit message conventions should align with those of the upstream project. <br>
          2. Minimize modifications to the upstream code as much as possible. Do not submit changes based on personal preferences or custom requirements for your own company, and do not attempt to “optimize” the upstream code.
      </td>
    </tr>
    <tr>
      <td>
        <div style="min-width: max-content; display: block; white-space: nowrap;">
          <a href="https://atomgit.com/Harmonybrew/homebrew-core">homebrew-core</a>
        </div>
      </td>
      <td>Formula Repository</td>
      <td>The vast majority of formulas are ported from upstream and can be considered forks of the upstream repository</td>
      <td>The packages users download are built from the formulas in this repository.</td>
      <td>This repository has its own documentation: <a href="./contribute-formula.md">How to Contribute Formulas</a></td>
    </tr>
    <tr>
      <td><a href="https://atomgit.com/Harmonybrew/ci">ci</a></td>
      <td>Pipeline scripts</td>
      <td>New</td>
      <td>
        Maintain pipeline scripts. Since we do not use GitHub Actions but instead use CodeArts Pipeline
        as our pipeline platform, we cannot directly reuse upstream pipeline scripts; we need to write our own scripts to implement
