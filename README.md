# b64_grep
Python Utility to grep things within encoded b64 data

When used as a program, it generates the 3 possible B64 Permutations for a given cleartext.  
Input can be given as argument or via stdin (convenient for POSIX bash uses). 
Output is designed to be feed into a regex engine, accordingly each possible value is separated by a pipe "|".  


Can also be used as a toolbox in other python scripts.

## CONTEXT
  Searching strings in base64 encoded data doesn't require decoding the said data.  
  However base 64 convert an input of 3 char at a time, to a 4 char output.  
  
      abc -> YWJj  
          a will influence byte[0] and byte[1] (YW). 
          b will influence byte[1] and byte[2] (WJ)  
          c will influence byte[2] and byte[3] (Jj)  
       
  Shifting the first letter to the right will change the output. 
  
     abc -> YWJj  
     zab(c)  -> emFi(Yw==)  
     zza(bc) -> enph(YmM=). 
  
  So there are 3 possible b64 depending of the offset of the starting char (a), those are caled "permutations".  
  Note that shift right produces some garbage output which needs to be removed.  
  
  In the same manner if the converted text is smaller than 3 caracters "\x00" padding is added (showing up as "=").  
  
      "   " -> AAA=  
      "c  " -> Yw==  
      "bc " -> YmM=  
      // " " is used to represent 'Null'       

  As stated above, input input_byte[1] and input_byte[2], influence output_bytes[1], output_bytes[2],output_bytes[3].  
  
      So when shifting right the input by 1 char, at the last round of the conversion, output_byte[1],output_byte[2],output_byte[3] must be dropped.  
      So when shifting right the input by 2 char, at the last round of the conversion, output_byte[2],output_byte[3] must be dropped.  
        
This program was dev using the follwing resources :  

  @Resource https://www.leeholmes.com/searching-for-content-in-base-64-strings/. 
  
  @Resource https://en.wikipedia.org/wiki/Base64   
  
  @Resource https://yara.readthedocs.io/en/stable/writingrules.html?highlight=base64#base64-strings  

# Licence
```C
/*
 * ----------------------------------------------------------------------------
 * "THE BEER-WARE LICENSE" (Revision 42):
 * <TWP_Zero> wrote this file.  As long as you retain this notice you
 * can do whatever you want with this stuff. If we meet some day, and you think
 * this stuff is worth it, you can buy me a beer in return.   
 * ----------------------------------------------------------------------------
 */
```
