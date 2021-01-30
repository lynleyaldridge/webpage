# workaround for loading both user and project .Rprofile files
# source: https://alison.rbind.io/post/2019-02-21-hugo-page-bundles/

if (file.exists("~/.Rprofile")) {
  base::sys.source("~/.Rprofile", envir = environment())
}

options(blogdown.author = "Lynley Aldridge",
        blogdown.ext = ".Rmd",
        blogdown.subdir = "post",
        blogdown.yaml.empty = TRUE,
        blogdown.new_bundle = TRUE,
        blogdown.title_case = TRUE)

