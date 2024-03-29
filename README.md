A+ enrollment questionnaire - template for common questions
===========================================================

This repository contains a template for an A+ enrollment questionnaire.
Enrollment questionnaire is a normal A+ questionnaire that students must
complete when they enroll in the course in A+.

How to use this in your course
------------------------------

1. Ensure that the course is using an up-to-date version of the a-plus-rst-tools
   (this template defines the questionnaire in RST markup).
   The version matching the git checksum `7e18045` (15th Aug 2019) is at least new enough.
2. Modify the `conf.py` file in the root of your course repository.
   Add these definitions for the `course_key` and `rst_prolog` variables as well as
   `exclude_patterns`:

   ```python
   # The JavaScript code used by the enrollment questionnaire is hosted in the course repo,
   # so we need to know the course key in order to craft the URL of the JS.
   # This RST substitution is used to insert the script in the questionnaire RST code.
   course_key = os.environ.get('COURSE_KEY', 'default')
   rst_prolog = '''.. |enroll-js-script| raw:: html

     <script src="/static/{course_key}/_static/enrollmentquiz.js"></script>
   '''.format(course_key=course_key)
   
   # Add 'enrollment' to the exclude_patterns list. The list has probably
   # initially been defined with some elements.
   exclude_patterns = ['_build', '_data', 'enrollment']
   ```
   
   The environment variable `COURSE_KEY` should be defined in `build.sh` for the
   production mooc-grader and in `docker-compile.sh` for local testing.
   In `build.sh`, you need to know the course key used in the production installation
   and export the variable like this: `export COURSE_KEY='mycoursekey'`.
   You can see the course key in A+ by looking at the import URL in the Edit course
   page: in the URL `https://grader.cs.aalto.fi/samplecourse/aplus-json`,
   the course key is `samplecourse`.
   In `docker-compile.sh`, add the parameter `-e "COURSE_KEY=default"` to the
   `docker run` command (the course key is `default` in the local testing container).

   If the `exclude_patterns` variable is not defined correctly, compiling the
   course RST fails with an error (because the question templates should not
   be compiled independently; they are included into another RST file):
   
   ```
   File "/path/to/course/a-plus-rst-tools/directives/questionnaire.py", line 152, in create_question
       env.question_count += 1
   AttributeError: 'BuildEnvironment' object has no attribute 'question_count'
   ```

3. Add this aplus-enrollment-questionnaire repository as a git submodule to
   the root of the course repository. The name of the directory inside the
   course repository shall be `enrollment`. You should not change the name
   `enrollment` since it has been hardcoded into `enrollment_en.rst` and
   `enrollment_fi.rst`. The submodule contains commonly used questions and files.

   ```
   git submodule add https://github.com/apluslms/aplus-enrollment-questionnaire.git enrollment
   ```

   If the git submodule has been added to your course git repository and you
   do not have it yet in your local repository in your computer, you need to run
   the command `git pull && git submodule update --init`.

4. You need to add the enrollment questionnaire to the course contents as follows.
   
   1. Modify the `index.rst` file of some exercise round (module).

      The first module is usually suitable for this purpose. The opening time
      of the module affects the enrollment exercise too in addition to the
      enrollment opening and closing times of the course, thus the module opening time
      should not be later than the course enrollment opening time.

      The module `index.rst` may actually have a different filename than `index.rst`
      since it is defined in the main `index.rst` of the course.

      The module index.rst defines the chapters of the module with
      the `toctree` directive. **Add** either the chapter `enrollment_en` or `enrollment_fi`
      (English or Finnish version) to the **end** of the first module in
      the module index.rst. **Note**: the opening time of the module
      affects the enrollment questionnaire too.

      If the course is open to external students, then add the chapter
      `enrollment_external_en` or `enrollment_external_fi` to the end of
      the module index.rst.

   2. **Copy** the corresponding file (`enrollment_en.rst` or `enrollment_fi.rst`)
      from the `enrollment` directory into the module directory. Similarly,
      `enrollment_external_en` or `enrollment_external_fi` if external students
      may enroll in the course.

   3. You may modify the questionnaire in that file if necessary.

5. Create a symbolic link pointing to the file `enrollment/_static/enrollmentquiz.js`
   in the path `_static/enrollmentquiz.js` so that the JS file is included
   in the static files of the course. (If there are issues with symbolic links,
   just copy the file.)

   ```bash
   cd _static
   ln -s ../enrollment/_static/enrollmentquiz.js enrollmentquiz.js
   ```

