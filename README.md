Based on : https://developers.redhat.com/blog/2019/11/18/new-tools-for-automating-end-to-end-tests-for-vs-code-extensions/



A solution at this link suggests  : 

1. Download the appropriate version of ChromeDriver, which meant knowing the version of Chromium packaged inside the Electron browser our VS Code uses.
2. Add ChromeDriver to our PATH.
3. Choose the appropriate VS Code binary
4. Set up our VS Code to run the tests properly. We cannot, for instance, use the native title bar.
5. Download another instance of VS Code just for testing. (We do not want to mess up the instance of VS Code we actually use.)
6. Build our extension.
7. Install the extension into the new instance.



Create hello world project to test. 



    npm install --save-dev vscode-extension-tester mocha @types/mocha
    npm install --save-dev chai @types/chai
    

 create file src/ui-test/helloworld-test.ts (in src so typescript can compile).

For test, we will use the files compiled to out.

Add script to run tests: 

    "ui-test": "npm run compile && extest setup-and-run out/ui-test/*.js"

Create test-resources

- this folder will contain
  - instance of vscode for testing
  - chromedriver binary
  - screenshots from failed tests
  - must be excluded from vsce packaging command (else errors)
  - add to excludes field in tsconfig.json
        "exclude": ["node_modules", ".vscode-test", "test-resources"]
  - update .vscodeignore to make sure it is excluded from vsix files
        test-resources/**

Write a test for workbench and run a command 

    import { Workbench, Notification, WebDriver, VSBrowser, NotificationType } from 'vscode-extension-tester';
    import { expect } from 'chai';
    
    describe('Hello World Example UI Tests', () => {
        let driver: WebDriver;
    
        before(() => {
            driver = VSBrowser.instance.driver;
        });
    
        it('Command shows a notification with the correct text', async () => {
            const workbench = new Workbench();
            await workbench.executeCommand('Hello World');
            const notification = await driver.wait(() => { return notificationExists('Hello'); }, 2000) as Notification;
    
            expect(await notification.getMessage()).equals('Hello World!');
            expect(await notification.getType()).equals(NotificationType.Info);
        });
    });
    
    // Get notification with text
    async function notificationExists(text: string): Promise<Notification | undefined> {
        const notifications = await new Workbench().getNotifications();
        for (const notification of notifications) {
            const message = await notification.getMessage();
            if (message.indexOf(text) >= 0) {
                return notification;
            }
        }
    }



    npm run ui-test





Resources : 

- https://developers.redhat.com/blog/2019/11/18/new-tools-for-automating-end-to-end-tests-for-vs-code-extensions/
- vscode extension tester : https://github.com/redhat-developer/vscode-extension-tester/wiki/Test-Setup#using-the-cli
- extension tester page objects: https://github.com/redhat-developer/vscode-extension-tester/wiki/Page-Object-APIs
Also try : https://vscode.rocks/testing/