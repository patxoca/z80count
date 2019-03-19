# z80count

This is a simple that parses Z80 assembler using regular expressions (I know!)
and add comments to the code with the cycles used by the instruction.

It needs testing and probably a proper Z80 parser, but it works for me and the
Z80 assembler syntax I use.

## Requirements

The tool requires Python 3.

## Usage

You can use it with:

    z80count.py  < file.asm > file_c.asm

Or inside `vim` you can:

    :% !z8count.py -su

With `-s` the tool adds a subtotal to the comments and `-u` tries to update
existing comments generated by the tool.

Example:
```
	push hl
	pop bc
	ld hl, $5800

	ld e, 7
.fade_out_all_loop0
	push hl
	push bc

	halt
.fade_out_all_loop1
	ld a, (hl)
	and 7
	jr z, no_fade_all_ink
	dec a
.no_fade_all_ink

	ld d, a

	ld a, (hl)
	and $38
	jr z, no_fade_all_paper
	sub 8
.no_fade_all_paper

	or d
	ld d, a

	ld a, (hl)
	and $c0
	or d

	ld (hl), a
	inc hl

	dec bc
	ld a, b
	or c
	jr nz, fade_out_all_loop1

	pop bc
	pop hl
	dec e
	jr nz, fade_out_all_loop0
```

Processed with `z80count.py -s` results in:
```
	push hl				; [11 .. 11/11]
	pop bc				; [10 .. 21/21]
	ld hl, $5800			; [10 .. 31/31]

	ld e, 7				; [7 .. 38/38]
.fade_out_all_loop0
	push hl				; [11 .. 49/49]
	push bc				; [11 .. 60/60]

	halt				; [4 .. 64/64]
.fade_out_all_loop1
	ld a, (hl)			; [7 .. 71/71]
	and 7				; [7 .. 78/78]
	jr z, no_fade_all_ink		; [12/7 .. 90/85]
	dec a				; [4 .. 94/89]
.no_fade_all_ink

	ld d, a				; [4 .. 98/93]

	ld a, (hl)			; [7 .. 105/100]
	and $38				; [7 .. 112/107]
	jr z, no_fade_all_paper		; [12/7 .. 124/114]
	sub 8				; [7 .. 131/121]
.no_fade_all_paper

	or d				; [4 .. 135/125]
	ld d, a				; [4 .. 139/129]

	ld a, (hl)			; [7 .. 146/136]
	and $c0				; [7 .. 153/143]
	or d				; [4 .. 157/147]

	ld (hl), a			; [7 .. 164/154]
	inc hl				; [6 .. 170/160]

	dec bc				; [6 .. 176/166]
	ld a, b				; [4 .. 180/170]
	or c				; [4 .. 184/174]
	jr nz, fade_out_all_loop1	; [12/7 .. 196/181]

	pop bc				; [10 .. 206/191]
	pop hl				; [10 .. 216/201]
	dec e				; [4 .. 220/205]
	jr nz, fade_out_all_loop0   	; [12/7 .. 232/212]
```

Comments show subtotals, and there are two types:
 - `[A .. T0/T1]`
 - `[B/A .. T0/T1]`

Where A, B, T0 and T1 are:
 - A is the number of cycles of current instruction. In case of a conditional
   instruction, this is the value when the condition is not met.
 - B is the number of cycles of current instruction when the condition is met.
 - T0 is the subtotal assuming all the conditionals were not met.
 - T1 is the subtotal assuming all the conditionals were met.

## Troubleshooting

Here be dragons!

Use `-d` flag if you think one instruction is not correctly parsed.

Feel free to open a PR if you find a bug!

