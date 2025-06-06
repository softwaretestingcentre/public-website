---
title: "Side Quest 3 - Testing Juice Shop locally"
date: 2025-06-06
---
While I was looking at the code for [OWASP's Juice Shop](https://github.com/juice-shop/juice-shop) I noticed it contains some Cypress tests, so lets try to run them locally.

To get `npm install` to work on my Ubuntu machine, I first had to:
```bash
nvm install v22.14.0
sudo apt-get install libgtk2.0-0 libgtk-3-0 libgbm-dev libnotify-dev libnss3 libxss1 libasound2 libxtst6 xauth xvfb
```

Because the app is meant to be vulnerable, there will be a lot of audit failures:
```
42 vulnerabilities (1 low, 15 moderate, 19 high, 7 critical)
```

Then run the app with `npm start`
```
> juice-shop@17.3.0 start
> node build/app

info: Detected Node.js version v22.14.0 (OK)
info: Detected OS linux (OK)
info: Detected CPU x64 (OK)
info: Configuration default validated (OK)
info: Entity models 19 of 19 are initialized (OK)
info: Required file server.js is present (OK)
info: Required file index.html is present (OK)
info: Required file styles.css is present (OK)
info: Required file main.js is present (OK)
info: Required file tutorial.js is present (OK)
info: Required file runtime.js is present (OK)
info: Required file vendor.js is present (OK)
info: Port 3000 is available (OK)
info: Chatbot training data botDefaultTrainingData.json validated (OK)
info: Domain https://www.alchemy.com/ is reachable (OK)
info: Server listening on port 3000
```

And we're up:

![image](https://github.com/user-attachments/assets/7e0c23bf-8225-4509-8926-d9d9c4ca343b)

Starting Cypress with `npx cypress open` we can see a range of tests already defined:

![image](https://github.com/user-attachments/assets/63c05318-29d1-4cef-b93c-a3a35b665e79)

Let's try `scoreboard.spec.ts`:
```typescript
describe('/#/score-board', () => {
  describe('challenge "scoreBoard"', () => {
    it('should be possible to access score board', () => {
      cy.visit('/#/score-board')
      cy.url().should('match', /\/score-board/)
      cy.expectChallengeSolved({ challenge: 'Score Board' })
    })
  })

  describe('challenge "continueCode"', () => {
    it('should be possible to solve the non-existent challenge #99', () => {
      cy.window().then(async () => {
        await fetch(
          `${Cypress.config('baseUrl')}/rest/continue-code/apply/69OxrZ8aJEgxONZyWoz1Dw4BvXmRGkM6Ae9M7k2rK63YpqQLPjnlb5V5LvDj`,
          {
            method: 'PUT',
            cache: 'no-cache',
            headers: {
              'Content-type': 'text/plain'
            }
          }
        )
      })
      cy.visit('/#/score-board')
      cy.expectChallengeSolved({ challenge: 'Imaginary Challenge' })
    })
  })
})
```

And when we run it, we see both test cases passing:

![image](https://github.com/user-attachments/assets/39a13cf7-3e10-45f6-b44d-32ee4a4ac40c)

![image](https://github.com/user-attachments/assets/be5350a2-1c9a-4c55-b031-886f91eb5bf8)

![image](https://github.com/user-attachments/assets/3b4dca4e-6eef-40f1-9ed8-4a8b5c7b6b98)
