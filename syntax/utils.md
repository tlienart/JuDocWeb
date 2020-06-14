# Utils

\blurb{The `utils.jl` file allows you to create your own functions to custom Franklin.}

\lineskip

In general Franklin supports two types of functions which get called at different stages in the rendering process.

\toc

## hfun

`hfun_` functions are called using `{{ command }}` and return HTML code i.e this would call `hfun_command()`. 
They can be used for various examples like listing the last posts on the main page.

This could look like this:

In `index.md`:
`{{ newest_posts }}`

In `utils.jl`:

```
function hfun_newest_posts()
    rpaths = String[]
    for file in keys(Franklin.ALL_PAGE_VARS)
        if startswith(file, "blog/")
            push!(rpaths, file)
        end
    end

    dates = Dates.Date[]
    for ref in rpaths
        push!(dates, pagevar(ref, "date"))
    end
    result = "&lt;div>"
    order = sortperm(dates; rev=true)
    for ref in rpaths[order]
        pos = findlast('/', ref)
        page_name = ref[1:pos-1]
        title = pagevar(ref, "title")
        desc = pagevar(ref, "desc")
        date = pagevar(ref, "date")
        str_date = Dates.format(date, "dd U YYYY")
        result *= "&lt;h3>&lt;u>&lt;a href='/$page_name'>$title&lt;/a>&lt;/u>&lt;/h3>"
        result *= "&lt;p class='post-list-desc'>$desc&lt;/p>"
    end
    result *= "&lt;/div>"
    return result
end
```

This creates a list of pages ordered by the page variable `date` and also shows the description of each post.

- `ALL_PAGE_VARS` stores all pages and their variables 
  - the keys look like `folder_path/index` if you have the structure `folder_path/index.md` for your posts
- `pagevar` takes two arguments: path of the page and the variable 
  
`hfun` functions can take arguments with:

`{{ newest_posts ARG1 ARG2}}` which could be used to define the sorting for example.
This would call `hfun_newest_posts(ARG1, ARG2)`.

## lx

`lx` functions can be used if one wants to return markdown code instead of `html`.

Currently they take only one argument which needs to be parsed in each function. A simple example is to define 
`lx_figclick` which should provide the same functionality as `figalt` but is a clickable image to view it in its full size.

These functions are written with the $\LaTeX$ like syntax: `\figclick{Arg}`.

In the example case it would look like `\figclick{alt, path}` and the function called:

```
html_img_click(src, alt) = "[![$(Franklin.htmlesc(alt))]($src)]($src)"

function lx_figclick(lx, _)
    # keep this first line
    brace_content = Franklin.content(lx.braces[1]) # input string
    alt, rpath = strip.(split(brace_content, ','))
    path  = Franklin.parse_rpath(rpath; canonical=false, code=true)
    return html_img_click(path, alt)
end
```
