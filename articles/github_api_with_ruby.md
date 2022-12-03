

Introduction

I've been using github repositories to save text data (e.g. interview questions, code snippets, etc.) for a while now, and they've always been pretty hard to maintain. So I asked myself, is there any way to automate this? Is it possible to create a CRUD application on ruby or rails and save all the data to a file inside a github repo instead of a database. Let's take a look.

Main 

I went through a bunch of articles and repositories to figure it out, but I couldn't find an easy way to do it. Yes, I know it may seem strange, but trust me, it was just that. So what do I mean when I say "easy way"? I'll try to explain. Let's try to imagine an interface that can meet our needs. What do we need? What are our requirements? So, first of all, we want to have the following features
- Add a new file with a content to the repo.
- Update the content of this file.
- Remove the file.

That's it. Pretty simple, isn't it? So what do we need to implement it? Let's list all the items. We need to have a `repo` name. What else? We don't want to add a new file to all repos with the same name, right? So we need to add a `user_name`. What else? Do we want to save some data to a file? Of course we do! The third item is `content`. Almost done. Do we have enough rights to commit something not in our repositories? Of course not, so what do we do? Right, add `personal_access_token` or `password`. That's it. Finally, all done, let's summarize and try to describe our interface. I my mind, interface may look like on the snipped below: 

```rb
# get
github_client.file_content(repo_name, user_name, file_name, personal_access_token)
# add
github_client.add_file(repo_name, user_name, file_name, content, personal_access_token)
# update 
github_client.update_file(repo_name, user_name, file_name, content, personal_access_token)
# delete 
github_client.delete(repo_name, user_name, file_name, personal_access_token)
```

Sounds simple, doesn't it? Is there any way to improve it? I think so, we have to repeat `personal_access_token` every time. Wouldn't it make sense to move it up a level?

```rb
github_client = GithubClient.new(personal_access_token)

# get
github_client.file_content(repo_name, user_name, file_name)
# add
github_client.add_file(repo_name, user_name, file_name, content)
# update 
github_client.update_file(repo_name, user_name, file_name, content)
# delete 
github_client.delete(repo_name, user_name, file_name)
```

So here it's. We have interface, let's find a gem which can do the same for us. After a short research I've found the following. gems:
- https://github.com/piotrmurach/github
- https://github.com/octokit/octokit.rb

Let's see if the first one can do what we need.

Install the gem by running

```rb
gem install github_api
```

And let's see if this gem can do something simple, like get a list with all repositories for some users. I expect to see an interface like this `github_client.repositories_list(user_name, api)`

And what we have:

```rb
require 'github_api'

puts Github.repos.list user: 'user_name'
```

Not bad, don't you think? Let's dive in further and really create something. First of all, we need to create your personal github access token that we'll use in our app ( unless you want to always use your password ). Here's a quick tutorial on how to create and get it  https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token

```rb
require 'github_api'

contents = Github::Client::Repos::Contents.new oauth_token: 'personal_github_token'

contents.create 'username', 'repo_name', 'file.md',
  path: 'file.md',
  message: 'Your commit message',
  content: 'The contents of your file'

```

And it works! Almost the same interface as we predicted. Only `path` and `message` weren't so obvious. My bad. Let't try to do the same for the update and delete actions. For these actions we need to find `file.sha` before any manipulation, don't worry, it's easy.

```rb
require 'github_api'

contents = Github::Client::Repos::Contents.new oauth_token: 'personal_github_token'
file = contents.find user: 'username', repo: 'repo_name', path: 'file.md'
```

And after that we can update or remove the file.


Update
```rb
contents.update 'username', 'repo_name', 'file.md',
  path: 'file.md',
  message: 'Your commit message',
  content: 'Updated content',
  file: file.sha
```

Delete
```rb
contents.delete 'username', 'repo_name', 'file.md',
  path: 'file.md',
  message: 'Your commit message',
  file: file.sha
```

Conclusion

Congratulations! Now you can save any content you want! Well, there is actually another gem `octokit.rb` mentioned earlier, but this one has almost the same features and looks more complex for our purposes to me. So, I guess we can summarize what we learned here:

- To interact with github we need to know 5 basic parameters: `repo_name, user_name, file_name, content and api_key`
- CRUD github content are not as complicated as we previously thought.
- There's a gem which allows us to do this easily https://github.com/piotrmurach/github
