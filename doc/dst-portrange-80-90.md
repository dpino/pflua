# dst portrange 80-90


## BPF

```
000: A = P[12:2]
001: if (A == 34525) goto 2 else goto 8
002: A = P[20:1]
003: if (A == 132) goto 6 else goto 4
004: if (A == 6) goto 6 else goto 5
005: if (A == 17) goto 6 else goto 20
006: A = P[56:2]
007: if (A >= 80) goto 18 else goto 20
008: if (A == 2048) goto 9 else goto 20
009: A = P[23:1]
010: if (A == 132) goto 13 else goto 11
011: if (A == 6) goto 13 else goto 12
012: if (A == 17) goto 13 else goto 20
013: A = P[20:2]
014: if (A & 8191 != 0) goto 20 else goto 15
015: X = (P[14:1] & 0xF) << 2
016: A = P[X+16:2]
017: if (A >= 80) goto 18 else goto 20
018: if (A > 90) goto 20 else goto 19
019: return 65535
020: return 0
```


## BPF cross-compiled to Lua

```
return function (P, length)
   local A = 0
   local X = 0
   local T = 0
   if 14 > length then return 0 end
   A = bit.bor(bit.lshift(P[12], 8), P[12+1])
   if not (A==34525) then goto L7 end
   if 21 > length then return 0 end
   A = P[20]
   if (A==132) then goto L5 end
   if (A==6) then goto L5 end
   if not (A==17) then goto L19 end
   ::L5::
   if 58 > length then return 0 end
   A = bit.bor(bit.lshift(P[56], 8), P[56+1])
   if (runtime_u32(A)>=80) then goto L17 end
   goto L19
   ::L7::
   if not (A==2048) then goto L19 end
   if 24 > length then return 0 end
   A = P[23]
   if (A==132) then goto L12 end
   if (A==6) then goto L12 end
   if not (A==17) then goto L19 end
   ::L12::
   if 22 > length then return 0 end
   A = bit.bor(bit.lshift(P[20], 8), P[20+1])
   if not (bit.band(A, 8191)==0) then goto L19 end
   if 14 >= length then return 0 end
   X = bit.lshift(bit.band(P[14], 15), 2)
   T = bit.tobit((X+16))
   if T < 0 or T + 2 > length then return 0 end
   A = bit.bor(bit.lshift(P[T], 8), P[T+1])
   if not (runtime_u32(A)>=80) then goto L19 end
   ::L17::
   if (runtime_u32(A)>90) then goto L19 end
   do return 65535 end
   ::L19::
   do return 0 end
   error("end of bpf")
end
```


## Direct pflang compilation

```
return function(P,length)
   if length < 34 then return false end
   local var1 = cast("uint16_t*", P+12)[0]
   if var1 == 8 then
      local var2 = P[23]
      if var2 == 6 then goto L8 end
      do
         if var2 == 17 then goto L8 end
         if var2 == 132 then goto L8 end
         return false
      end
::L8::
      if band(cast("uint16_t*", P+20)[0],65311) ~= 0 then return false end
      local var9 = lshift(band(P[14],15),2)
      if (var9 + 18) > length then return false end
      local var16 = rshift(bswap(cast("uint16_t*", P+(var9 + 16))[0]), 16)
      if var16 < 80 then return false end
      return var16 <= 90
   else
      if length < 58 then return false end
      if var1 ~= 56710 then return false end
      local var24 = P[20]
      if var24 == 6 then goto L24 end
      do
         if var24 ~= 44 then goto L27 end
         do
            if P[54] == 6 then goto L24 end
            goto L27
         end
::L27::
         if var24 == 17 then goto L24 end
         if var24 ~= 44 then goto L33 end
         do
            if P[54] == 17 then goto L24 end
            goto L33
         end
::L33::
         if var24 == 132 then goto L24 end
         if var24 ~= 44 then return false end
         if P[54] == 132 then goto L24 end
         return false
      end
::L24::
      local var34 = rshift(bswap(cast("uint16_t*", P+56)[0]), 16)
      if var34 < 80 then return false end
      return var34 <= 90
   end
end

```

