# Enabling GitHub Sync | GitBook Documentation

#### Getting started

In the space you want to sync with your GitHub repo, head to the [space header](https://gitbook.com/docs/resources/gitbook-ui#space-header) in the top right, and select **Configure**. From the provider list, select **GitHub Sync**.

![A GitBook screenshot showing GitHub Sync configuration options](https://gitbook.com/docs/~gitbook/image?url=https%3A%2F%2F1050631731-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FNkEGS7hzeqa35sMXQZ4X%252Fuploads%252FSmP9bdDqDK0gOpDYdSdj%252FEnabling%2520GitHub%2520Sync%25402x.png%3Falt%3Dmedia%26token%3Da10a9f24-4f69-4b34-a802-64424aac7f76\&width=768\&dpr=4\&quality=100\&sign=53baed17\&sv=2)

GitHub Sync configuration options.

#### Authenticate with GitHub

If you’re setting up GitHub Sync for the first time and haven’t already linked a GitHub account, you’ll be prompted to do that when you begin configuring Git Sync. If you’ve already linked your account, you may still need to authenticate via GitHub.

{% hint style="warning" %}
If you see a **"Potential duplicated accounts"** error message at this step, your GitHub account is already linked with another GitBook user account.

To identify which accounts are linked, log out of this session and sign in using the **Sign in with GitHub** method. If you already know the GitBook account associated with GitHub, log into that user account and unlink your GitHub account (done in settings) before logging back in and linking your current account.

Read more on our [troubleshooting page](https://gitbook.com/docs/getting-started/git-sync/troubleshooting#potential-duplicated-accounts-when-signing-in).
{% endhint %}

#### Install the GitBook app to your GitHub account

If you haven’t already done so, you’ll see a prompt to add the [GitBook apparrow-up-right](https://github.com/apps/gitbook-com) to your GitHub account.

Follow the instructions in the GitHub popover and either give GitBook specific repository permissions, or allow access to all repositories, depending on your needs.

#### Select a repository and branch

Select the account and repository you want to keep in sync with your GitBook content.

{% hint style="info" %}
Can’t see your repository? If you can't find your repository in the list, make sure that you've installed the [GitBook GitHub apparrow-up-right](https://github.com/apps/gitbook-com) in the right scope (i.e. your personal account or the GitHub org where the repository lives). You should also check that you’ve configured the correct repository access in the GitBook GitHub app.
{% endhint %}

Once you’ve selected the correct repository, choose which branch you want commits to be pushed to and synced from.

#### Perform an initial sync

{% stepper %}
{% step %}
### GitBook → GitHub

Sync your space’s content to the selected branch. This is useful if you’re starting from an empty repository and want to get your GitBook content in quickly.
{% endstep %}

{% step %}
### GitHub → GitBook

Sync your space’s content from the selected branch. This is useful if you have existing Markdown content in a repository and want to bring it into GitBook.
{% endstep %}
{% endstepper %}

#### Write and commit

You’re good to go. If your space was in [live edit](https://gitbook.com/docs/collaboration/live-edits) mode, live edits are now locked. This allows reliable sync when someone on your team merges a [change request](https://gitbook.com/docs/collaboration/change-requests) in GitBook.

* When you edit on GitBook, every change request merge will result in a commit to your selected GitHub branch.
* When you commit to GitHub, every commit will be synced to your GitBook space as a history commit.

{% hint style="warning" %}
The GitHub app that powers our GitHub integration is currently not available to customers on GitHub Enterprise Server instances.
{% endhint %}

Last updated 13 days ago
