<h2><strong>Getting Started with CircleCI and CrossBrowserTesting</strong></h2>
<a href="https://circleci.com/">CircleCI</a> is a continuous integration tool that lets you automate your development process quickly, safely, and at scale. Through CircleCI's integration with GitHub, GitHub Enterprise, and Bitbucket, every time you commit code, a build is created and automatically run in a clean container or virtual machine, allowing you to test every commit.

In this guide we will use CircleCI with Github for testing using the Selenium Webdriver and Python programming language.
<h3>Setting up CircleCI</h3>
1. Navigate to your GitHub account and <a href="https://github.com/new">create a new repository</a>.
<h4><img src="http://help.crossbrowsertesting.com/wp-content/uploads/2019/01/create_repository-2.png" /></h4>
2. Add file tests/test_selenium.py Don't forget to set your username and authkey, which can be found under Accounts and Settings, in that file. Don't have a CrossBrowserTesting account? Start a free trial at https://crossbrowsertesting.com/ and start running your Selenium tests in the cloud.
Or visit https://help.crossbrowsertesting.com/selenium-testing/getting-started/cbthelper-php/

```
python
import unittest
from selenium import webdriver
import requests
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait

class LoginForm(unittest.TestCase):
    def setUp(self):

        # Put your username and authkey below
        # You can find your authkey at crossbrowsertesting.com/account
        self.username = "YOUR_USERNAME"
        self.authkey  = "YOUR_AUTHKEY"

        self.api_session = requests.Session()
        self.api_session.auth = (self.username,self.authkey)

        self.test_result = None

        caps = {}

        caps['name'] = 'Screenshot Example'
        caps['browserName'] = 'Chrome'
        caps['version'] = '71x64'
        caps['platform'] = 'Windows 10'
        caps['screenResolution'] = '1366x768'

        self.driver = webdriver.Remote(
            desired_capabilities=caps,
            command_executor="http://%s:%s@hub.crossbrowsertesting.com:80/wd/hub"%(self.username,self.authkey)
        )

        self.driver.implicitly_wait(20)

    def test_CBT(self):

        try:
            self.driver.get('http://crossbrowsertesting.github.io/login-form.html')
            self.driver.maximize_window()
            self.driver.find_element_by_name('username').send_keys('tester@crossbrowsertesting.com')
            self.driver.find_element_by_name('password').send_keys('test123')
            self.driver.find_element_by_css_selector('body &gt; div &gt; div &gt; div &gt; div &gt; form &gt; div.form-actions &gt; button').click()

            elem = WebDriverWait(self.driver, 10).until(
                EC.presence_of_element_located((By.XPATH, '//*[@id=\"logged-in-message\"]/h2'))
            )

            welcomeText = elem.text
            self.assertEqual("Welcome tester@crossbrowsertesting.com", welcomeText)

            print("Taking snapshot")
            snapshot_hash = self.api_session.post('https://crossbrowsertesting.com/api/v3/selenium/' + self.driver.session_id + '/snapshots').json()['hash']

            self.test_result = 'pass'

        except AssertionError as e:
            # log the error message, and set the score to "during tearDown()".
            self.api_session.put('https://crossbrowsertesting.com/api/v3/selenium/' + self.driver.session_id + '/snapshots/' + snapshot_hash,
                data={'description':"AssertionError: " + str(e)})
            self.test_result = 'fail'
            raise

    def tearDown(self):
        print("Done with session %s" % self.driver.session_id)
        self.driver.quit()
        # Here we make the api call to set the test's score.
        # Pass it it passes, fail if an assertion fails, unset if the test didn't finish
        if self.test_result is not None:
            self.api_session.put('https://crossbrowsertesting.com/api/v3/selenium/' + self.driver.session_id,
                data={'action':'set_score', 'score':self.test_result})

if __name__ == '__main__':
    unittest.main()
```
&nbsp;

3. Add file .circleci/config.yml to the new repository
<pre><code>version: 2
jobs:
  build:
    docker:
      - image: circleci/python:jessie-node-browsers
    steps:
      - checkout
      - run: mkdir test-reports
      - run: sudo pip install selenium
      - run: sudo pip install requests
      - run: 
          command: python tests/test_selenium.py
</code></pre>
&nbsp;
<h4><strong>Running Your Build on CircleCI</strong></h4>
1. From the <a href="https://circleci.com/dashboard">CircleCI Dashboard</a>, after selecting the <strong>Add Projects</strong> tab, click the Set Up Project button next to your desired repository.

<img src="http://help.crossbrowsertesting.com/wp-content/uploads/2019/01/set_up_project.png" />

2. Scroll to the bottom of the page and click the <strong>Start Building</strong> button.

<img src="http://help.crossbrowsertesting.com/wp-content/uploads/2019/01/Start_building_button.png" />

You should see your build start to run in CircleCI and in the Crossbrowsertesting app <a href="https://app.crossbrowsertesting.com/selenium/results">here</a>.

If you have any questions or concerns, feel <a href="mailto:support@crossbrowsertesting.com">free to get in touch</a>.

