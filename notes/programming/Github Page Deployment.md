When deploying a github page in a subdirectory that is **NOT** the actual page repo itself, such as:

```https://user/testpagedeployment ```, and not ```https://user/user.github.io/```,

Github pages start the directory architecture from the actual github repo page itself, taking it as root. 

If we assume that the filestructure in the repo "testpagedeployment" is:
```
testpagedeployment
    mainsource
        index.html
    test.png
```

To be able to insert test.png in the index.html file, you need to call it via:

```/testpagedeployment/test.png```

As the above code is equal to:

`https://user/user.github.io/testpagedeployment/test.png`

which *should* be a valid path. 
