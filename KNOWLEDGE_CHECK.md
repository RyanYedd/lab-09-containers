# Lab 09: Knowledge check

**Answer each question below it, in your own words, in one to three sentences.** Replace the `...` below **Your answer:** with your text.

## 1. Credentials in the image?

Assume you want to share your image with others and make it publicly available on `ghcr.io`, so that anyone can pull it and run the analysis. A teammate suggests adding `COPY .aws /home/app/.aws` to the `Dockerfile`, so that nobody has to type `-v ~/.aws:/home/app/.aws:ro` any more. (This is a thought experiment: keep your own package private.)

**Your answer:** Give two reasons why this is a bad idea. Think of who could then read your credentials, and of what happens when your Learner Lab session ends.

Anyone who can pull the public Docker image could access my AWS credentials and MongoDB passwords, which would expose sensitive information. Also, Docker keeps files in image layers even if they are deleted later. When my Learner Lab session ends, my AWS credentials expire, so storing them in the image would also leave outdated credentials. Credentials should be mounted at runtime instead.

## 2. Where did the Atlas password come from?

In Step 4, you ran the analysis once with `-v ~/.aws:/home/app/.aws:ro` only, without the three `-e` options. The container still wrote to your Atlas cluster.

*Hint:* Read `get_atlas_settings()` in `src/book_analysis/analyze.py`.

**Your answer:** Explain, step by step, where the container found the Atlas settings in that run.

Because I did not pass the three MongoDB environment variables with -e, the container used the fallback in get_atlas_settings(). It accessed AWS using the credentials mounted from my computer in ~/.aws and retrieved the mongodb_credentials parameter from AWS Systems Manager Parameter Store. That parameter contained the Atlas settings, including the password, which allowed the container to connect to MongoDB.

## 3. Same tag, different image

Run `docker images book-analysis`, and note the image ID of `book-analysis:0.1.0`. Then change the `description` in the `LABEL` of your `Dockerfile`, build again with the same command, `docker build -t book-analysis:0.1.0 .`, and run `docker images book-analysis` again.

**Your answer:** Paste the image ID from before and after the rebuild. Why is it a problem for someone who uses your image that `0.1.0` now names a different image, and what should you have done instead?

Before rebuilding, my image ID was 12c909494cde. After changing the description and rebuilding, my new image ID was 9d04e398c54e. This is a problem because someone using version 0.1.0 expects the same image every time, but the tag now points to a different image. I should create a new version tag, such as 0.1.1, instead of changing an image that was already published as 0.1.0.
