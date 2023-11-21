
**Next.js 13**


1. ssr can not receive browser events like click, access browser API, maintain store, use effect

2. by default under the app folder all components will have server-side rendering

3. 'use client top of the component will make client component'

4. if we use 'use client' then all the dependent components will be client component

5. use as much server-side rendering as keep client-side less

6. client-side rendering has a large bundle, is resource intensive, less secure, has no SEO, extra roundtrip to server

7. caching is only available in fetch() function

8. use no-store cache for dynamic data fetching on every request other wise it will be static data fetching
    ![image](https://github.com/iamsohel/necessary-resources/assets/9135426/fc117cf8-ea39-4d54-a601-bbe5107ed2fc)


