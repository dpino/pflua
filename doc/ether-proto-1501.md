# ether proto 1501


## BPF

```
000: A = P[12:2]
001: if (A == 1501) goto 2 else goto 3
002: return 65535
003: return 0
```


## BPF cross-compiled to Lua

```
return function (P, length)
   local A = 0
   if 14 > length then return 0 end
   A = bit.bor(bit.lshift(P[12], 8), P[12+1])
   if not (A==1501) then goto L2 end
   do return 65535 end
   ::L2::
   do return 0 end
   error("end of bpf")
end
```


## Direct pflang compilation

```
return function(P,length)
   if length < 14 then return false end
   return cast("uint16_t*", P+12)[0] == 56581
end

```

