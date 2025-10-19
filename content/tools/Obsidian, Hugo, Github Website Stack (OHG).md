First, you'll need to create a github account. If you won't be using a custom domain (You can add one later), make sure you name the account what you want your website to be. The domain will be {{ account-name.github.io }}

Once you have an account, the first thing you will do is generate a new ssh key and add it to github. This will give you passwordless access to your github account from your desktop. From my Fedora Linux machine, I will run:  
`ssh-keygen -t ed25519 -C "exampleemail@email.com"`

When it asks for the name of the key, give it a unique name to use for this account. I just picked the gitup account name to make it easy to identify:  
`id_ed25519_example`

Then, grab your public key from ~/.ssh/id_ed25519_example.pub and add it in github under settings > SSH and GPG Keys > New SSH Key

![](../images/Pasted%20image%2020251019034622.png)

If you have multiple github accounts, you'll have to create a couple aliases under ~/.ssh/config:
```bash
Host github.com
   HostName github.com
   IdentityFile ~/.ssh/id_ed25519
   IdentitiesOnly yes

# Other github account: everythingelsexyz
Host github-everythingelsexyz
   HostName github.com
   IdentityFile ~/.ssh/id_ed25519_everythingelsexyz
   IdentitiesOnly yes
```

Then add your private keys to the ssh agent:
```bash
ssh-add ~/.ssh/id_ed25519
ssh-add ~/.ssh/id_ed25519_everythingelsexyz
```

Then add the username and email settings while in the new project directory:
```bash
everythingelsexyz.github.io on  main 
❯ git config user.name "everythingelsexyz"

everythingelsexyz.github.io on  main 
❯ git config user.email "example@email.com"
```

Next, go to your github homepage and select "Create repository". The repository name needs to match your github account name from earlier. It also must have public visibility:

![](../images/Pasted%20image%2020251019034937.png)

From your desktop, create a directory with {{ reponame.github.io }}. You must use this format for this to work:
`mkdir everythingelsexyz.github.io && cd everythingelsexyz.github.io`

Follow the instructions on github for "…or create a new repository on the command line"
```bash
echo "# everythingelsexyz" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/everythingelsexyz/everythingelsexyz.git
git push -u origin main
git remote set-url origin git@github-everythingelsexyz:everythingelsexyz/everythingelsexyz.git

```

Host


Go through quick setup here: https://pages.github.com/

Hugo Setup: https://gohugo.io/hosting-and-deployment/hosting-on-github/

Here's how to set up a free website using Github pages and Hugo. You can add a custom domain name using [Cloudflare](https://www.cloudflare.com/) for about $10 per year. 

The general steps include:
1. Create a Github repository for your site. 
2. Change your pages settings.
3. Build the repo into a Hugo site. 
4. Add DNS records for a custom domain. 
