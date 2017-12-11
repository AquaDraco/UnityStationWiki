To test network interactions, you should set up a realistic environment.

bla bla bla, todo
...

## Set up ping simulation
To simulate network latency you need to select `Network Manager` in Hierarchy tab:

![](https://image.prntscr.com/image/ErpN3x62SfK3bcTzsjE5Zw.png)

Then find `Custom Network Manager` script in Inspector tab and tick `Use network simulation`:

![](https://image.prntscr.com/image/9lknAZmmR021gW8pCeNhsQ.png)

Set up desirable ping (200 is recommended) and save (so that you don't lose that setting next time Unity crashes)!

![](https://image.prntscr.com/image/QGX1GsbWTxqxOlEOPKs3wg.png)

Note that actual ping will be more than expected, `200 ping` setting usually makes it 250-750