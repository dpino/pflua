# src net ffff:ffff:eeee:eeee:0:0:0:0/72


## BPF

```
000: A = P[12:2]
001: if (A == 34525) goto 2 else goto 9
002: A = P[22:4]
003: if (A == 4294967295) goto 4 else goto 9
004: A = P[26:4]
005: if (A == 4008636142) goto 6 else goto 9
006: A = P[30:4]
007: if (A & 4278190080 != 0) goto 9 else goto 8
008: return 65535
009: return 0
```


## BPF cross-compiled to Lua

```
return function (P, length)
   local A = 0
   if 14 > length then return 0 end
   A = bit.bor(bit.lshift(P[12], 8), P[12+1])
   if not (A==34525) then goto L8 end
   if 26 > length then return 0 end
   A = bit.bor(bit.lshift(P[22], 24),bit.lshift(P[22+1], 16), bit.lshift(P[22+2], 8), P[22+3])
   if not (A==-1) then goto L8 end
   if 30 > length then return 0 end
   A = bit.bor(bit.lshift(P[26], 24),bit.lshift(P[26+1], 16), bit.lshift(P[26+2], 8), P[26+3])
   if not (A==-286331154) then goto L8 end
   if 34 > length then return 0 end
   A = bit.bor(bit.lshift(P[30], 24),bit.lshift(P[30+1], 16), bit.lshift(P[30+2], 8), P[30+3])
   if not (bit.band(A, -16777216)==0) then goto L8 end
   do return 65535 end
   ::L8::
   do return 0 end
   error("end of bpf")
end
```


## Direct pflang compilation

```
return function(P,length)
   if length < 54 then return false end
   if cast("uint16_t*", P+12)[0] ~= 56710 then return false end
   if cast("uint32_t*", P+22)[0] ~= 4294967295 then return false end
   if cast("uint32_t*", P+26)[0] ~= 4008636142 then return false end
   return band(cast("uint32_t*", P+30)[0],255) == 0
end

```

