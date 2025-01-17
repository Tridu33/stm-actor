[SSAM: A Haskell Parallel Programming STM
Based Simple Actor Model](https://iopscience.iop.org/article/10.1088/1742-6596/1566/1/012040/pdf)
# stm-actor

[![Hackage](https://img.shields.io/hackage/v/stm-actor.svg)](https://hackage.haskell.org/package/stm-actor)

An implementation of a basic actor model in Haskell. With a very simple API,
this is meant to serve as a basis for writing simple, message-passing style
of programs. Here is an example using the `etcd` library.

```haskell
{-# LANGUAGE BlockArguments, LambdaCase, OverloadedStrings #-}
import Network.Etcd

main :: IO ()
main = do
  client <- createClient ["machine-1", "machine-2", "machine-3"]
  logger <- act . forever $ do
    receive \changes -> do
      -- we can do arbitrary things here with the reported changes, of course
      liftIO (appendFile "logfile" (show changes))
  watcher <- act do
    link logger
    liftIO $ forever do
      waitClient client "cluster-resources" >>= \case
        Nothing -> pure ()
        Just updatedNode -> atomically (send actor updatedNode)
```

If you want multi-node actors or you care about throughput, this
is not the package for you. The design is optimized to have low latency on
message receipt, and to allow for interactions between actors in transactions, using
Haskell's software transactional memory.
