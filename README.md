# RISCV_Automatic-Sanitizer-Dispenser
This github repository summarizes the progress made in the ASIC class for the riscv_project.

## Aim
The project's goal is to design an automatic sanitizer dispenser utilizing a specialized RISCV processor for bottle refill alerts and touch-free sanitizer distribution, with the objective of minimizing the machine's footprint, energy consumption, and overall expenses.

## Working
An infrared sensor is employed to detect the presence of a hand. Upon hand detection, the solenoid valve is activated. The sanitizer is directed through the flow sensor via the open solenoid valve, allowing the flow sensor to calculate the quantity of sanitizer dispensed. The system is configured to accommodate a sanitizer bottle with a capacity of half a liter. Upon detecting the dispensing of half a liter, the flow sensor triggers a buzzer, signaling the need for a sanitizer bottle refill.

## Circuit
This is not the depiction of the final circuit. <br>

![ckt](ckt_depict.png)
## Block Diagram
![blockd](bda.png)

## C Code
```
void read();
void operate(int);
void bottle_status();

int ir_sen_ip;
int solenoid_valve_op = 0;
int flow_sensor_ip;
int led_op = 0;
int buzzer = 0;
int counter_h = 0;
int counter_l = 0;
int reset_button = 0;
float water_used=0;

int main()
{ 
    int solenoid_reg, led_reg, buzzer_reg;
    solenoid_reg = solenoid_valve_op*4;
    led_reg = led_op*8;
    buzzer_reg = buzzer*16;
    asm(
	"or x30, x30, %0\n\t"
	"or x30, x30, %1\n\t"
	"or x30, x30, %2\n\t" 
	:"=r"(solenoid_reg),"=r"(led_reg),"=r"(buzzer_reg));
		
    while(1){
    //reset = digital_read(0);
    	asm(
	"andi %0, x30, 1\n\t"
	:"=r"(reset_button));
    if(reset_button)
    {
        solenoid_valve_op = 0;
        led_op = 0;
        buzzer = 0;
        counter_h = 0;
        counter_l = 0;
        water_used = 0;
        //digital_write(4,solenoid_valve_op);
        //digital_write(5,led_op);
        //digital_write(6,buzzer);
        solenoid_reg = solenoid_valve_op*4;
        led_reg = led_op*8;
        buzzer_reg = buzzer*16;
        asm(
	"or x30, x30,%0 \n\t"
	"or x30, x30,%1 \n\t"
	"or x30, x30,%2 \n\t" 
	:"=r"(solenoid_reg),"=r"(led_reg),"=r"(buzzer_reg));
    }
    else
    {
    while(!led_op && !buzzer && !reset_button){
        read();

    }
    }
    }
    return 0;
}


void read()
{
   int ir_sen_ip_reg;
    //ir_sen_ip = digital_read(1);
    asm(
	"andi %0, x30, 2\n\t"
	:"=r"(ir_sen_ip_reg));
    ir_sen_ip = ir_sen_ip_reg/2;
    operate(ir_sen_ip);
}

void operate(int ir_value)
{
    int solenoid_reg;
    if(ir_value)
    {
    	
        solenoid_valve_op = 1;
        solenoid_reg = solenoid_valve_op*4;
        asm(
	"or x30, x30,%0 \n\t"
	:"=r"(solenoid_reg));
        //digital_write(4,solenoid_valve_op);
        counter_h++;
        bottle_status();
        

    }
    else {
        solenoid_valve_op = 0;
        //digital_write(4,solenoid_valve_op);
        solenoid_reg = solenoid_valve_op*4;
        asm(
	"or x30, x30,%0 \n\t"
	:"=r"(solenoid_reg));
        counter_l++;
    }

}

void bottle_status()
{
    int time = counter_h + counter_l;
    float freq = 1000000/time;
    float water = freq/7.5;
    float ls = water/60;
    int led_reg;
    int buzzer_reg;
    water_used = water_used+ls;
    if(ls==0.5)
    {
        led_op = 1;
        buzzer = 1;   
        //digital_write(5,led_op);
        //digital_write(6,buzzer);
        led_reg = led_op*8;
        buzzer_reg = buzzer*16;
        
        asm(
	"or x30, x30, %0\n\t"
	"or x30, x30, %1\n\t" 
	:"=r"(led_reg),	"=r"(buzzer_reg));
        
        
    }
}
```

## Assembly Code
```

san_disp.o:     file format elf32-littleriscv


Disassembly of section .text:

00010074 <register_fini>:
   10074:	ffff0797          	auipc	a5,0xffff0
   10078:	f8c78793          	addi	a5,a5,-116 # 0 <register_fini-0x10074>
   1007c:	00078863          	beqz	a5,1008c <register_fini+0x18>
   10080:	00001517          	auipc	a0,0x1
   10084:	cd450513          	addi	a0,a0,-812 # 10d54 <__libc_fini_array>
   10088:	4850006f          	j	10d0c <atexit>
   1008c:	00008067          	ret

00010090 <_start>:
   10090:	00003197          	auipc	gp,0x3
   10094:	9e818193          	addi	gp,gp,-1560 # 12a78 <__global_pointer$>
   10098:	c4018513          	addi	a0,gp,-960 # 126b8 <_edata>
   1009c:	c8018613          	addi	a2,gp,-896 # 126f8 <__BSS_END__>
   100a0:	40a60633          	sub	a2,a2,a0
   100a4:	00000593          	li	a1,0
   100a8:	5a1000ef          	jal	ra,10e48 <memset>
   100ac:	00001517          	auipc	a0,0x1
   100b0:	ca850513          	addi	a0,a0,-856 # 10d54 <__libc_fini_array>
   100b4:	459000ef          	jal	ra,10d0c <atexit>
   100b8:	4fd000ef          	jal	ra,10db4 <__libc_init_array>
   100bc:	00012503          	lw	a0,0(sp)
   100c0:	00410593          	addi	a1,sp,4
   100c4:	00000613          	li	a2,0
   100c8:	16c000ef          	jal	ra,10234 <main>
   100cc:	4550006f          	j	10d20 <exit>

000100d0 <__do_global_dtors_aux>:
   100d0:	c5c1c783          	lbu	a5,-932(gp) # 126d4 <completed.5434>
   100d4:	04079463          	bnez	a5,1011c <__do_global_dtors_aux+0x4c>
   100d8:	ffff0797          	auipc	a5,0xffff0
   100dc:	f2878793          	addi	a5,a5,-216 # 0 <register_fini-0x10074>
   100e0:	02078863          	beqz	a5,10110 <__do_global_dtors_aux+0x40>
   100e4:	ff010113          	addi	sp,sp,-16
   100e8:	00002517          	auipc	a0,0x2
   100ec:	18050513          	addi	a0,a0,384 # 12268 <__FRAME_END__>
   100f0:	00112623          	sw	ra,12(sp)
   100f4:	00000097          	auipc	ra,0x0
   100f8:	000000e7          	jalr	zero # 0 <register_fini-0x10074>
   100fc:	00c12083          	lw	ra,12(sp)
   10100:	00100793          	li	a5,1
   10104:	c4f18e23          	sb	a5,-932(gp) # 126d4 <completed.5434>
   10108:	01010113          	addi	sp,sp,16
   1010c:	00008067          	ret
   10110:	00100793          	li	a5,1
   10114:	c4f18e23          	sb	a5,-932(gp) # 126d4 <completed.5434>
   10118:	00008067          	ret
   1011c:	00008067          	ret

00010120 <frame_dummy>:
   10120:	ffff0797          	auipc	a5,0xffff0
   10124:	ee078793          	addi	a5,a5,-288 # 0 <register_fini-0x10074>
   10128:	00078c63          	beqz	a5,10140 <frame_dummy+0x20>
   1012c:	c6018593          	addi	a1,gp,-928 # 126d8 <object.5439>
   10130:	00002517          	auipc	a0,0x2
   10134:	13850513          	addi	a0,a0,312 # 12268 <__FRAME_END__>
   10138:	00000317          	auipc	t1,0x0
   1013c:	00000067          	jr	zero # 0 <register_fini-0x10074>
   10140:	00008067          	ret

00010144 <bottle_status>:
   10144:	ff010113          	addi	sp,sp,-16
   10148:	00112623          	sw	ra,12(sp)
   1014c:	00812423          	sw	s0,8(sp)
   10150:	00912223          	sw	s1,4(sp)
   10154:	c4c1a583          	lw	a1,-948(gp) # 126c4 <counter_h>
   10158:	c481a783          	lw	a5,-952(gp) # 126c0 <counter_l>
   1015c:	00f585b3          	add	a1,a1,a5
   10160:	000f4537          	lui	a0,0xf4
   10164:	24050513          	addi	a0,a0,576 # f4240 <__global_pointer$+0xe17c8>
   10168:	2a5000ef          	jal	ra,10c0c <__divsi3>
   1016c:	175000ef          	jal	ra,10ae0 <__floatsisf>
   10170:	c281a583          	lw	a1,-984(gp) # 126a0 <__SDATA_BEGIN__>
   10174:	578000ef          	jal	ra,106ec <__divsf3>
   10178:	c2c1a583          	lw	a1,-980(gp) # 126a4 <__SDATA_BEGIN__+0x4>
   1017c:	570000ef          	jal	ra,106ec <__divsf3>
   10180:	00050413          	mv	s0,a0
   10184:	00050593          	mv	a1,a0
   10188:	c401a503          	lw	a0,-960(gp) # 126b8 <_edata>
   1018c:	134000ef          	jal	ra,102c0 <__addsf3>
   10190:	c4a1a023          	sw	a0,-960(gp) # 126b8 <_edata>
   10194:	c301a583          	lw	a1,-976(gp) # 126a8 <__SDATA_BEGIN__+0x8>
   10198:	00040513          	mv	a0,s0
   1019c:	0d9000ef          	jal	ra,10a74 <__eqsf2>
   101a0:	00051863          	bnez	a0,101b0 <bottle_status+0x6c>
   101a4:	00100793          	li	a5,1
   101a8:	c4f1aa23          	sw	a5,-940(gp) # 126cc <led_op>
   101ac:	c4f1a823          	sw	a5,-944(gp) # 126c8 <buzzer>
   101b0:	00c12083          	lw	ra,12(sp)
   101b4:	00812403          	lw	s0,8(sp)
   101b8:	00412483          	lw	s1,4(sp)
   101bc:	01010113          	addi	sp,sp,16
   101c0:	00008067          	ret

000101c4 <operate>:
   101c4:	00051c63          	bnez	a0,101dc <operate+0x18>
   101c8:	c401ac23          	sw	zero,-936(gp) # 126d0 <solenoid_valve_op>
   101cc:	c481a783          	lw	a5,-952(gp) # 126c0 <counter_l>
   101d0:	00178793          	addi	a5,a5,1
   101d4:	c4f1a423          	sw	a5,-952(gp) # 126c0 <counter_l>
   101d8:	00008067          	ret
   101dc:	ff010113          	addi	sp,sp,-16
   101e0:	00112623          	sw	ra,12(sp)
   101e4:	00100713          	li	a4,1
   101e8:	c4e1ac23          	sw	a4,-936(gp) # 126d0 <solenoid_valve_op>
   101ec:	c4c1a783          	lw	a5,-948(gp) # 126c4 <counter_h>
   101f0:	00178793          	addi	a5,a5,1
   101f4:	c4f1a623          	sw	a5,-948(gp) # 126c4 <counter_h>
   101f8:	f4dff0ef          	jal	ra,10144 <bottle_status>
   101fc:	00c12083          	lw	ra,12(sp)
   10200:	01010113          	addi	sp,sp,16
   10204:	00008067          	ret

00010208 <read>:
   10208:	ff010113          	addi	sp,sp,-16
   1020c:	00112623          	sw	ra,12(sp)
   10210:	002f7793          	andi	a5,t5,2
   10214:	01f7d513          	srli	a0,a5,0x1f
   10218:	00f50533          	add	a0,a0,a5
   1021c:	40155513          	srai	a0,a0,0x1
   10220:	c6a1ac23          	sw	a0,-904(gp) # 126f0 <ir_sen_ip>
   10224:	fa1ff0ef          	jal	ra,101c4 <operate>
   10228:	00c12083          	lw	ra,12(sp)
   1022c:	01010113          	addi	sp,sp,16
   10230:	00008067          	ret

00010234 <main>:
   10234:	fd010113          	addi	sp,sp,-48
   10238:	02112623          	sw	ra,44(sp)
   1023c:	02812423          	sw	s0,40(sp)
   10240:	02912223          	sw	s1,36(sp)
   10244:	03212023          	sw	s2,32(sp)
   10248:	01312e23          	sw	s3,28(sp)
   1024c:	01412c23          	sw	s4,24(sp)
   10250:	01512a23          	sw	s5,20(sp)
   10254:	01612823          	sw	s6,16(sp)
   10258:	01712623          	sw	s7,12(sp)
   1025c:	01812423          	sw	s8,8(sp)
   10260:	001f7993          	andi	s3,t5,1
   10264:	00098a13          	mv	s4,s3
   10268:	0200006f          	j	10288 <main+0x54>
   1026c:	c401ac23          	sw	zero,-936(gp) # 126d0 <solenoid_valve_op>
   10270:	c401aa23          	sw	zero,-940(gp) # 126cc <led_op>
   10274:	c401a823          	sw	zero,-944(gp) # 126c8 <buzzer>
   10278:	c401a623          	sw	zero,-948(gp) # 126c4 <counter_h>
   1027c:	c401a423          	sw	zero,-952(gp) # 126c0 <counter_l>
   10280:	00000793          	li	a5,0
   10284:	c4f1a023          	sw	a5,-960(gp) # 126b8 <_edata>
   10288:	c541a223          	sw	s4,-956(gp) # 126bc <reset_button>
   1028c:	fe0990e3          	bnez	s3,1026c <main+0x38>
   10290:	c541a783          	lw	a5,-940(gp) # 126cc <led_op>
   10294:	c501a703          	lw	a4,-944(gp) # 126c8 <buzzer>
   10298:	00e7e7b3          	or	a5,a5,a4
   1029c:	fe0796e3          	bnez	a5,10288 <main+0x54>
   102a0:	f69ff0ef          	jal	ra,10208 <read>
   102a4:	c541a783          	lw	a5,-940(gp) # 126cc <led_op>
   102a8:	c501a703          	lw	a4,-944(gp) # 126c8 <buzzer>
   102ac:	00e7e7b3          	or	a5,a5,a4
   102b0:	c441a703          	lw	a4,-956(gp) # 126bc <reset_button>
   102b4:	00e7e7b3          	or	a5,a5,a4
   102b8:	fe0784e3          	beqz	a5,102a0 <main+0x6c>
   102bc:	fcdff06f          	j	10288 <main+0x54>

000102c0 <__addsf3>:
   102c0:	00800737          	lui	a4,0x800
   102c4:	ff010113          	addi	sp,sp,-16
   102c8:	fff70713          	addi	a4,a4,-1 # 7fffff <__global_pointer$+0x7ed587>
   102cc:	00a777b3          	and	a5,a4,a0
   102d0:	00912223          	sw	s1,4(sp)
   102d4:	01212023          	sw	s2,0(sp)
   102d8:	01f55493          	srli	s1,a0,0x1f
   102dc:	01755913          	srli	s2,a0,0x17
   102e0:	0175d513          	srli	a0,a1,0x17
   102e4:	00b77733          	and	a4,a4,a1
   102e8:	0ff97913          	andi	s2,s2,255
   102ec:	0ff57513          	andi	a0,a0,255
   102f0:	00112623          	sw	ra,12(sp)
   102f4:	00812423          	sw	s0,8(sp)
   102f8:	01f5d593          	srli	a1,a1,0x1f
   102fc:	00379793          	slli	a5,a5,0x3
   10300:	00371713          	slli	a4,a4,0x3
   10304:	40a906b3          	sub	a3,s2,a0
   10308:	18b49463          	bne	s1,a1,10490 <__addsf3+0x1d0>
   1030c:	08d05c63          	blez	a3,103a4 <__addsf3+0xe4>
   10310:	04051c63          	bnez	a0,10368 <__addsf3+0xa8>
   10314:	34070663          	beqz	a4,10660 <__addsf3+0x3a0>
   10318:	fff68693          	addi	a3,a3,-1
   1031c:	02069e63          	bnez	a3,10358 <__addsf3+0x98>
   10320:	00e787b3          	add	a5,a5,a4
   10324:	00090513          	mv	a0,s2
   10328:	00579713          	slli	a4,a5,0x5
   1032c:	10075c63          	bgez	a4,10444 <__addsf3+0x184>
   10330:	00150513          	addi	a0,a0,1
   10334:	0ff00713          	li	a4,255
   10338:	32e50e63          	beq	a0,a4,10674 <__addsf3+0x3b4>
   1033c:	7e000737          	lui	a4,0x7e000
   10340:	0017f693          	andi	a3,a5,1
   10344:	fff70713          	addi	a4,a4,-1 # 7dffffff <__global_pointer$+0x7dfed587>
   10348:	0017d793          	srli	a5,a5,0x1
   1034c:	00e7f7b3          	and	a5,a5,a4
   10350:	00d7e7b3          	or	a5,a5,a3
   10354:	0f00006f          	j	10444 <__addsf3+0x184>
   10358:	0ff00613          	li	a2,255
   1035c:	00c91e63          	bne	s2,a2,10378 <__addsf3+0xb8>
   10360:	0ff00513          	li	a0,255
   10364:	0e00006f          	j	10444 <__addsf3+0x184>
   10368:	0ff00613          	li	a2,255
   1036c:	fec90ae3          	beq	s2,a2,10360 <__addsf3+0xa0>
   10370:	04000637          	lui	a2,0x4000
   10374:	00c76733          	or	a4,a4,a2
   10378:	01b00613          	li	a2,27
   1037c:	00d65663          	bge	a2,a3,10388 <__addsf3+0xc8>
   10380:	00100713          	li	a4,1
   10384:	f9dff06f          	j	10320 <__addsf3+0x60>
   10388:	02000613          	li	a2,32
   1038c:	00d755b3          	srl	a1,a4,a3
   10390:	40d606b3          	sub	a3,a2,a3
   10394:	00d71733          	sll	a4,a4,a3
   10398:	00e03733          	snez	a4,a4
   1039c:	00e5e733          	or	a4,a1,a4
   103a0:	f81ff06f          	j	10320 <__addsf3+0x60>
   103a4:	06068663          	beqz	a3,10410 <__addsf3+0x150>
   103a8:	41250633          	sub	a2,a0,s2
   103ac:	02091463          	bnez	s2,103d4 <__addsf3+0x114>
   103b0:	00078e63          	beqz	a5,103cc <__addsf3+0x10c>
   103b4:	fff60613          	addi	a2,a2,-1 # 3ffffff <__global_pointer$+0x3fed587>
   103b8:	00061663          	bnez	a2,103c4 <__addsf3+0x104>
   103bc:	00e787b3          	add	a5,a5,a4
   103c0:	f69ff06f          	j	10328 <__addsf3+0x68>
   103c4:	0ff00693          	li	a3,255
   103c8:	00d51e63          	bne	a0,a3,103e4 <__addsf3+0x124>
   103cc:	00070793          	mv	a5,a4
   103d0:	0740006f          	j	10444 <__addsf3+0x184>
   103d4:	0ff00693          	li	a3,255
   103d8:	fed50ae3          	beq	a0,a3,103cc <__addsf3+0x10c>
   103dc:	040006b7          	lui	a3,0x4000
   103e0:	00d7e7b3          	or	a5,a5,a3
   103e4:	01b00693          	li	a3,27
   103e8:	00c6d663          	bge	a3,a2,103f4 <__addsf3+0x134>
   103ec:	00100793          	li	a5,1
   103f0:	fcdff06f          	j	103bc <__addsf3+0xfc>
   103f4:	02000693          	li	a3,32
   103f8:	40c686b3          	sub	a3,a3,a2
   103fc:	00c7d5b3          	srl	a1,a5,a2
   10400:	00d797b3          	sll	a5,a5,a3
   10404:	00f037b3          	snez	a5,a5
   10408:	00f5e7b3          	or	a5,a1,a5
   1040c:	fb1ff06f          	j	103bc <__addsf3+0xfc>
   10410:	00190693          	addi	a3,s2,1
   10414:	0fe6f513          	andi	a0,a3,254
   10418:	06051063          	bnez	a0,10478 <__addsf3+0x1b8>
   1041c:	04091263          	bnez	s2,10460 <__addsf3+0x1a0>
   10420:	fa0786e3          	beqz	a5,103cc <__addsf3+0x10c>
   10424:	02070063          	beqz	a4,10444 <__addsf3+0x184>
   10428:	00e787b3          	add	a5,a5,a4
   1042c:	00579713          	slli	a4,a5,0x5
   10430:	00075a63          	bgez	a4,10444 <__addsf3+0x184>
   10434:	fc000737          	lui	a4,0xfc000
   10438:	fff70713          	addi	a4,a4,-1 # fbffffff <__global_pointer$+0xfbfed587>
   1043c:	00e7f7b3          	and	a5,a5,a4
   10440:	00100513          	li	a0,1
   10444:	0077f713          	andi	a4,a5,7
   10448:	22070863          	beqz	a4,10678 <__addsf3+0x3b8>
   1044c:	00f7f713          	andi	a4,a5,15
   10450:	00400693          	li	a3,4
   10454:	22d70263          	beq	a4,a3,10678 <__addsf3+0x3b8>
   10458:	00478793          	addi	a5,a5,4
   1045c:	21c0006f          	j	10678 <__addsf3+0x3b8>
   10460:	20078463          	beqz	a5,10668 <__addsf3+0x3a8>
   10464:	ee070ee3          	beqz	a4,10360 <__addsf3+0xa0>
   10468:	00000493          	li	s1,0
   1046c:	020007b7          	lui	a5,0x2000
   10470:	0ff00513          	li	a0,255
   10474:	2040006f          	j	10678 <__addsf3+0x3b8>
   10478:	0ff00613          	li	a2,255
   1047c:	1ec68a63          	beq	a3,a2,10670 <__addsf3+0x3b0>
   10480:	00e787b3          	add	a5,a5,a4
   10484:	0017d793          	srli	a5,a5,0x1
   10488:	00068513          	mv	a0,a3
   1048c:	fb9ff06f          	j	10444 <__addsf3+0x184>
   10490:	08d05863          	blez	a3,10520 <__addsf3+0x260>
   10494:	04051863          	bnez	a0,104e4 <__addsf3+0x224>
   10498:	00090513          	mv	a0,s2
   1049c:	fa0704e3          	beqz	a4,10444 <__addsf3+0x184>
   104a0:	fff68693          	addi	a3,a3,-1 # 3ffffff <__global_pointer$+0x3fed587>
   104a4:	02069263          	bnez	a3,104c8 <__addsf3+0x208>
   104a8:	40e787b3          	sub	a5,a5,a4
   104ac:	00579713          	slli	a4,a5,0x5
   104b0:	f8075ae3          	bgez	a4,10444 <__addsf3+0x184>
   104b4:	04000437          	lui	s0,0x4000
   104b8:	fff40413          	addi	s0,s0,-1 # 3ffffff <__global_pointer$+0x3fed587>
   104bc:	0087f433          	and	s0,a5,s0
   104c0:	00050913          	mv	s2,a0
   104c4:	13c0006f          	j	10600 <__addsf3+0x340>
   104c8:	0ff00613          	li	a2,255
   104cc:	0ff00513          	li	a0,255
   104d0:	f6c90ae3          	beq	s2,a2,10444 <__addsf3+0x184>
   104d4:	01b00613          	li	a2,27
   104d8:	02d65263          	bge	a2,a3,104fc <__addsf3+0x23c>
   104dc:	00100713          	li	a4,1
   104e0:	0340006f          	j	10514 <__addsf3+0x254>
   104e4:	0ff00613          	li	a2,255
   104e8:	0ff00513          	li	a0,255
   104ec:	f4c90ce3          	beq	s2,a2,10444 <__addsf3+0x184>
   104f0:	04000637          	lui	a2,0x4000
   104f4:	00c76733          	or	a4,a4,a2
   104f8:	fddff06f          	j	104d4 <__addsf3+0x214>
   104fc:	02000613          	li	a2,32
   10500:	00d755b3          	srl	a1,a4,a3
   10504:	40d606b3          	sub	a3,a2,a3
   10508:	00d71733          	sll	a4,a4,a3
   1050c:	00e03733          	snez	a4,a4
   10510:	00e5e733          	or	a4,a1,a4
   10514:	40e787b3          	sub	a5,a5,a4
   10518:	00090513          	mv	a0,s2
   1051c:	f91ff06f          	j	104ac <__addsf3+0x1ec>
   10520:	06068a63          	beqz	a3,10594 <__addsf3+0x2d4>
   10524:	41250633          	sub	a2,a0,s2
   10528:	02091863          	bnez	s2,10558 <__addsf3+0x298>
   1052c:	02078063          	beqz	a5,1054c <__addsf3+0x28c>
   10530:	fff60613          	addi	a2,a2,-1 # 3ffffff <__global_pointer$+0x3fed587>
   10534:	00061863          	bnez	a2,10544 <__addsf3+0x284>
   10538:	40f707b3          	sub	a5,a4,a5
   1053c:	00058493          	mv	s1,a1
   10540:	f6dff06f          	j	104ac <__addsf3+0x1ec>
   10544:	0ff00693          	li	a3,255
   10548:	02d51063          	bne	a0,a3,10568 <__addsf3+0x2a8>
   1054c:	00070793          	mv	a5,a4
   10550:	00058493          	mv	s1,a1
   10554:	ef1ff06f          	j	10444 <__addsf3+0x184>
   10558:	0ff00693          	li	a3,255
   1055c:	fed508e3          	beq	a0,a3,1054c <__addsf3+0x28c>
   10560:	040006b7          	lui	a3,0x4000
   10564:	00d7e7b3          	or	a5,a5,a3
   10568:	01b00693          	li	a3,27
   1056c:	00c6d663          	bge	a3,a2,10578 <__addsf3+0x2b8>
   10570:	00100793          	li	a5,1
   10574:	fc5ff06f          	j	10538 <__addsf3+0x278>
   10578:	02000693          	li	a3,32
   1057c:	40c686b3          	sub	a3,a3,a2
   10580:	00c7d833          	srl	a6,a5,a2
   10584:	00d797b3          	sll	a5,a5,a3
   10588:	00f037b3          	snez	a5,a5
   1058c:	00f867b3          	or	a5,a6,a5
   10590:	fa9ff06f          	j	10538 <__addsf3+0x278>
   10594:	00190513          	addi	a0,s2,1
   10598:	0fe57513          	andi	a0,a0,254
   1059c:	04051863          	bnez	a0,105ec <__addsf3+0x32c>
   105a0:	02091c63          	bnez	s2,105d8 <__addsf3+0x318>
   105a4:	00079a63          	bnez	a5,105b8 <__addsf3+0x2f8>
   105a8:	fa0712e3          	bnez	a4,1054c <__addsf3+0x28c>
   105ac:	00000793          	li	a5,0
   105b0:	00000493          	li	s1,0
   105b4:	0c40006f          	j	10678 <__addsf3+0x3b8>
   105b8:	e80706e3          	beqz	a4,10444 <__addsf3+0x184>
   105bc:	40e786b3          	sub	a3,a5,a4
   105c0:	00569613          	slli	a2,a3,0x5
   105c4:	40f707b3          	sub	a5,a4,a5
   105c8:	f80644e3          	bltz	a2,10550 <__addsf3+0x290>
   105cc:	fe0680e3          	beqz	a3,105ac <__addsf3+0x2ec>
   105d0:	00068793          	mv	a5,a3
   105d4:	e71ff06f          	j	10444 <__addsf3+0x184>
   105d8:	e80796e3          	bnez	a5,10464 <__addsf3+0x1a4>
   105dc:	e80706e3          	beqz	a4,10468 <__addsf3+0x1a8>
   105e0:	00070793          	mv	a5,a4
   105e4:	00058493          	mv	s1,a1
   105e8:	d79ff06f          	j	10360 <__addsf3+0xa0>
   105ec:	40e78433          	sub	s0,a5,a4
   105f0:	00541693          	slli	a3,s0,0x5
   105f4:	0406d463          	bgez	a3,1063c <__addsf3+0x37c>
   105f8:	40f70433          	sub	s0,a4,a5
   105fc:	00058493          	mv	s1,a1
   10600:	00040513          	mv	a0,s0
   10604:	6bc000ef          	jal	ra,10cc0 <__clzsi2>
   10608:	ffb50513          	addi	a0,a0,-5
   1060c:	00a41433          	sll	s0,s0,a0
   10610:	03254e63          	blt	a0,s2,1064c <__addsf3+0x38c>
   10614:	41250533          	sub	a0,a0,s2
   10618:	00150513          	addi	a0,a0,1
   1061c:	02000713          	li	a4,32
   10620:	00a457b3          	srl	a5,s0,a0
   10624:	40a70533          	sub	a0,a4,a0
   10628:	00a41433          	sll	s0,s0,a0
   1062c:	00803433          	snez	s0,s0
   10630:	0087e7b3          	or	a5,a5,s0
   10634:	00000513          	li	a0,0
   10638:	e0dff06f          	j	10444 <__addsf3+0x184>
   1063c:	fc0412e3          	bnez	s0,10600 <__addsf3+0x340>
   10640:	00000793          	li	a5,0
   10644:	00000513          	li	a0,0
   10648:	f69ff06f          	j	105b0 <__addsf3+0x2f0>
   1064c:	fc0007b7          	lui	a5,0xfc000
   10650:	fff78793          	addi	a5,a5,-1 # fbffffff <__global_pointer$+0xfbfed587>
   10654:	40a90533          	sub	a0,s2,a0
   10658:	00f477b3          	and	a5,s0,a5
   1065c:	de9ff06f          	j	10444 <__addsf3+0x184>
   10660:	00090513          	mv	a0,s2
   10664:	de1ff06f          	j	10444 <__addsf3+0x184>
   10668:	00070793          	mv	a5,a4
   1066c:	cf5ff06f          	j	10360 <__addsf3+0xa0>
   10670:	0ff00513          	li	a0,255
   10674:	00000793          	li	a5,0
   10678:	00579713          	slli	a4,a5,0x5
   1067c:	00075e63          	bgez	a4,10698 <__addsf3+0x3d8>
   10680:	00150513          	addi	a0,a0,1
   10684:	0ff00713          	li	a4,255
   10688:	04e50e63          	beq	a0,a4,106e4 <__addsf3+0x424>
   1068c:	fc000737          	lui	a4,0xfc000
   10690:	fff70713          	addi	a4,a4,-1 # fbffffff <__global_pointer$+0xfbfed587>
   10694:	00e7f7b3          	and	a5,a5,a4
   10698:	0ff00713          	li	a4,255
   1069c:	0037d793          	srli	a5,a5,0x3
   106a0:	00e51863          	bne	a0,a4,106b0 <__addsf3+0x3f0>
   106a4:	00078663          	beqz	a5,106b0 <__addsf3+0x3f0>
   106a8:	004007b7          	lui	a5,0x400
   106ac:	00000493          	li	s1,0
   106b0:	0ff57513          	andi	a0,a0,255
   106b4:	00c12083          	lw	ra,12(sp)
   106b8:	00812403          	lw	s0,8(sp)
   106bc:	00979793          	slli	a5,a5,0x9
   106c0:	01751713          	slli	a4,a0,0x17
   106c4:	0097d513          	srli	a0,a5,0x9
   106c8:	01f49493          	slli	s1,s1,0x1f
   106cc:	00e56533          	or	a0,a0,a4
   106d0:	00956533          	or	a0,a0,s1
   106d4:	00012903          	lw	s2,0(sp)
   106d8:	00412483          	lw	s1,4(sp)
   106dc:	01010113          	addi	sp,sp,16
   106e0:	00008067          	ret
   106e4:	00000793          	li	a5,0
   106e8:	fb1ff06f          	j	10698 <__addsf3+0x3d8>

000106ec <__divsf3>:
   106ec:	fd010113          	addi	sp,sp,-48
   106f0:	02912223          	sw	s1,36(sp)
   106f4:	01755493          	srli	s1,a0,0x17
   106f8:	01312e23          	sw	s3,28(sp)
   106fc:	01512a23          	sw	s5,20(sp)
   10700:	01612823          	sw	s6,16(sp)
   10704:	00951a93          	slli	s5,a0,0x9
   10708:	02112623          	sw	ra,44(sp)
   1070c:	02812423          	sw	s0,40(sp)
   10710:	03212023          	sw	s2,32(sp)
   10714:	01412c23          	sw	s4,24(sp)
   10718:	01712623          	sw	s7,12(sp)
   1071c:	01812423          	sw	s8,8(sp)
   10720:	01912223          	sw	s9,4(sp)
   10724:	0ff4f493          	andi	s1,s1,255
   10728:	00058b13          	mv	s6,a1
   1072c:	009ada93          	srli	s5,s5,0x9
   10730:	01f55993          	srli	s3,a0,0x1f
   10734:	08048663          	beqz	s1,107c0 <__divsf3+0xd4>
   10738:	0ff00793          	li	a5,255
   1073c:	0af48263          	beq	s1,a5,107e0 <__divsf3+0xf4>
   10740:	003a9a93          	slli	s5,s5,0x3
   10744:	040007b7          	lui	a5,0x4000
   10748:	00faeab3          	or	s5,s5,a5
   1074c:	f8148493          	addi	s1,s1,-127
   10750:	00000b93          	li	s7,0
   10754:	017b5513          	srli	a0,s6,0x17
   10758:	009b1413          	slli	s0,s6,0x9
   1075c:	0ff57513          	andi	a0,a0,255
   10760:	00945413          	srli	s0,s0,0x9
   10764:	01fb5b13          	srli	s6,s6,0x1f
   10768:	08050c63          	beqz	a0,10800 <__divsf3+0x114>
   1076c:	0ff00793          	li	a5,255
   10770:	0af50863          	beq	a0,a5,10820 <__divsf3+0x134>
   10774:	00341413          	slli	s0,s0,0x3
   10778:	040007b7          	lui	a5,0x4000
   1077c:	00f46433          	or	s0,s0,a5
   10780:	f8150513          	addi	a0,a0,-127
   10784:	00000793          	li	a5,0
   10788:	002b9713          	slli	a4,s7,0x2
   1078c:	00f76733          	or	a4,a4,a5
   10790:	fff70713          	addi	a4,a4,-1
   10794:	00e00693          	li	a3,14
   10798:	0169c933          	xor	s2,s3,s6
   1079c:	40a48a33          	sub	s4,s1,a0
   107a0:	0ae6e063          	bltu	a3,a4,10840 <__divsf3+0x154>
   107a4:	00001697          	auipc	a3,0x1
   107a8:	98868693          	addi	a3,a3,-1656 # 1112c <__errno+0xc>
   107ac:	00271713          	slli	a4,a4,0x2
   107b0:	00d70733          	add	a4,a4,a3
   107b4:	00072703          	lw	a4,0(a4)
   107b8:	00d70733          	add	a4,a4,a3
   107bc:	00070067          	jr	a4
   107c0:	020a8a63          	beqz	s5,107f4 <__divsf3+0x108>
   107c4:	000a8513          	mv	a0,s5
   107c8:	4f8000ef          	jal	ra,10cc0 <__clzsi2>
   107cc:	ffb50793          	addi	a5,a0,-5
   107d0:	f8a00493          	li	s1,-118
   107d4:	00fa9ab3          	sll	s5,s5,a5
   107d8:	40a484b3          	sub	s1,s1,a0
   107dc:	f75ff06f          	j	10750 <__divsf3+0x64>
   107e0:	0ff00493          	li	s1,255
   107e4:	00200b93          	li	s7,2
   107e8:	f60a86e3          	beqz	s5,10754 <__divsf3+0x68>
   107ec:	00300b93          	li	s7,3
   107f0:	f65ff06f          	j	10754 <__divsf3+0x68>
   107f4:	00000493          	li	s1,0
   107f8:	00100b93          	li	s7,1
   107fc:	f59ff06f          	j	10754 <__divsf3+0x68>
   10800:	02040a63          	beqz	s0,10834 <__divsf3+0x148>
   10804:	00040513          	mv	a0,s0
   10808:	4b8000ef          	jal	ra,10cc0 <__clzsi2>
   1080c:	ffb50793          	addi	a5,a0,-5
   10810:	00f41433          	sll	s0,s0,a5
   10814:	f8a00793          	li	a5,-118
   10818:	40a78533          	sub	a0,a5,a0
   1081c:	f69ff06f          	j	10784 <__divsf3+0x98>
   10820:	0ff00513          	li	a0,255
   10824:	00200793          	li	a5,2
   10828:	f60400e3          	beqz	s0,10788 <__divsf3+0x9c>
   1082c:	00300793          	li	a5,3
   10830:	f59ff06f          	j	10788 <__divsf3+0x9c>
   10834:	00000513          	li	a0,0
   10838:	00100793          	li	a5,1
   1083c:	f4dff06f          	j	10788 <__divsf3+0x9c>
   10840:	00541b13          	slli	s6,s0,0x5
   10844:	128af663          	bgeu	s5,s0,10970 <__divsf3+0x284>
   10848:	fffa0a13          	addi	s4,s4,-1
   1084c:	00000493          	li	s1,0
   10850:	010b5b93          	srli	s7,s6,0x10
   10854:	00010437          	lui	s0,0x10
   10858:	000b8593          	mv	a1,s7
   1085c:	fff40413          	addi	s0,s0,-1 # ffff <register_fini-0x75>
   10860:	000a8513          	mv	a0,s5
   10864:	3b0000ef          	jal	ra,10c14 <__udivsi3>
   10868:	008b7433          	and	s0,s6,s0
   1086c:	00050593          	mv	a1,a0
   10870:	00050c93          	mv	s9,a0
   10874:	00040513          	mv	a0,s0
   10878:	370000ef          	jal	ra,10be8 <__mulsi3>
   1087c:	00050c13          	mv	s8,a0
   10880:	000b8593          	mv	a1,s7
   10884:	000a8513          	mv	a0,s5
   10888:	3d4000ef          	jal	ra,10c5c <__umodsi3>
   1088c:	01051513          	slli	a0,a0,0x10
   10890:	0104d493          	srli	s1,s1,0x10
   10894:	00a4e533          	or	a0,s1,a0
   10898:	000c8993          	mv	s3,s9
   1089c:	01857e63          	bgeu	a0,s8,108b8 <__divsf3+0x1cc>
   108a0:	01650533          	add	a0,a0,s6
   108a4:	fffc8993          	addi	s3,s9,-1
   108a8:	01656863          	bltu	a0,s6,108b8 <__divsf3+0x1cc>
   108ac:	01857663          	bgeu	a0,s8,108b8 <__divsf3+0x1cc>
   108b0:	ffec8993          	addi	s3,s9,-2
   108b4:	01650533          	add	a0,a0,s6
   108b8:	418504b3          	sub	s1,a0,s8
   108bc:	000b8593          	mv	a1,s7
   108c0:	00048513          	mv	a0,s1
   108c4:	350000ef          	jal	ra,10c14 <__udivsi3>
   108c8:	00050593          	mv	a1,a0
   108cc:	00050c13          	mv	s8,a0
   108d0:	00040513          	mv	a0,s0
   108d4:	314000ef          	jal	ra,10be8 <__mulsi3>
   108d8:	00050a93          	mv	s5,a0
   108dc:	000b8593          	mv	a1,s7
   108e0:	00048513          	mv	a0,s1
   108e4:	378000ef          	jal	ra,10c5c <__umodsi3>
   108e8:	01051513          	slli	a0,a0,0x10
   108ec:	000c0413          	mv	s0,s8
   108f0:	01557e63          	bgeu	a0,s5,1090c <__divsf3+0x220>
   108f4:	01650533          	add	a0,a0,s6
   108f8:	fffc0413          	addi	s0,s8,-1
   108fc:	01656863          	bltu	a0,s6,1090c <__divsf3+0x220>
   10900:	01557663          	bgeu	a0,s5,1090c <__divsf3+0x220>
   10904:	ffec0413          	addi	s0,s8,-2
   10908:	01650533          	add	a0,a0,s6
   1090c:	01099993          	slli	s3,s3,0x10
   10910:	41550533          	sub	a0,a0,s5
   10914:	0089e9b3          	or	s3,s3,s0
   10918:	00a03533          	snez	a0,a0
   1091c:	00a9e433          	or	s0,s3,a0
   10920:	07fa0713          	addi	a4,s4,127
   10924:	0ee05663          	blez	a4,10a10 <__divsf3+0x324>
   10928:	00747793          	andi	a5,s0,7
   1092c:	00078a63          	beqz	a5,10940 <__divsf3+0x254>
   10930:	00f47793          	andi	a5,s0,15
   10934:	00400693          	li	a3,4
   10938:	00d78463          	beq	a5,a3,10940 <__divsf3+0x254>
   1093c:	00440413          	addi	s0,s0,4
   10940:	00441793          	slli	a5,s0,0x4
   10944:	0007da63          	bgez	a5,10958 <__divsf3+0x26c>
   10948:	f80007b7          	lui	a5,0xf8000
   1094c:	fff78793          	addi	a5,a5,-1 # f7ffffff <__global_pointer$+0xf7fed587>
   10950:	00f47433          	and	s0,s0,a5
   10954:	080a0713          	addi	a4,s4,128
   10958:	0fe00793          	li	a5,254
   1095c:	00345413          	srli	s0,s0,0x3
   10960:	04e7d463          	bge	a5,a4,109a8 <__divsf3+0x2bc>
   10964:	00000413          	li	s0,0
   10968:	0ff00713          	li	a4,255
   1096c:	03c0006f          	j	109a8 <__divsf3+0x2bc>
   10970:	01fa9493          	slli	s1,s5,0x1f
   10974:	001ada93          	srli	s5,s5,0x1
   10978:	ed9ff06f          	j	10850 <__divsf3+0x164>
   1097c:	00098913          	mv	s2,s3
   10980:	000a8413          	mv	s0,s5
   10984:	000b8793          	mv	a5,s7
   10988:	00200713          	li	a4,2
   1098c:	fce78ce3          	beq	a5,a4,10964 <__divsf3+0x278>
   10990:	00300713          	li	a4,3
   10994:	0ce78863          	beq	a5,a4,10a64 <__divsf3+0x378>
   10998:	00100713          	li	a4,1
   1099c:	f8e792e3          	bne	a5,a4,10920 <__divsf3+0x234>
   109a0:	00000413          	li	s0,0
   109a4:	00000713          	li	a4,0
   109a8:	00941413          	slli	s0,s0,0x9
   109ac:	0ff77713          	andi	a4,a4,255
   109b0:	01771713          	slli	a4,a4,0x17
   109b4:	00945413          	srli	s0,s0,0x9
   109b8:	01f91513          	slli	a0,s2,0x1f
   109bc:	00e46433          	or	s0,s0,a4
   109c0:	00a46533          	or	a0,s0,a0
   109c4:	02c12083          	lw	ra,44(sp)
   109c8:	02812403          	lw	s0,40(sp)
   109cc:	02412483          	lw	s1,36(sp)
   109d0:	02012903          	lw	s2,32(sp)
   109d4:	01c12983          	lw	s3,28(sp)
   109d8:	01812a03          	lw	s4,24(sp)
   109dc:	01412a83          	lw	s5,20(sp)
   109e0:	01012b03          	lw	s6,16(sp)
   109e4:	00c12b83          	lw	s7,12(sp)
   109e8:	00812c03          	lw	s8,8(sp)
   109ec:	00412c83          	lw	s9,4(sp)
   109f0:	03010113          	addi	sp,sp,48
   109f4:	00008067          	ret
   109f8:	000b0913          	mv	s2,s6
   109fc:	f8dff06f          	j	10988 <__divsf3+0x29c>
   10a00:	00400437          	lui	s0,0x400
   10a04:	00000913          	li	s2,0
   10a08:	00300793          	li	a5,3
   10a0c:	f7dff06f          	j	10988 <__divsf3+0x29c>
   10a10:	00100793          	li	a5,1
   10a14:	40e787b3          	sub	a5,a5,a4
   10a18:	01b00713          	li	a4,27
   10a1c:	f8f742e3          	blt	a4,a5,109a0 <__divsf3+0x2b4>
   10a20:	09ea0513          	addi	a0,s4,158
   10a24:	00f457b3          	srl	a5,s0,a5
   10a28:	00a41433          	sll	s0,s0,a0
   10a2c:	00803433          	snez	s0,s0
   10a30:	0087e433          	or	s0,a5,s0
   10a34:	00747793          	andi	a5,s0,7
   10a38:	00078a63          	beqz	a5,10a4c <__divsf3+0x360>
   10a3c:	00f47793          	andi	a5,s0,15
   10a40:	00400713          	li	a4,4
   10a44:	00e78463          	beq	a5,a4,10a4c <__divsf3+0x360>
   10a48:	00440413          	addi	s0,s0,4 # 400004 <__global_pointer$+0x3ed58c>
   10a4c:	00541793          	slli	a5,s0,0x5
   10a50:	00345413          	srli	s0,s0,0x3
   10a54:	f407d8e3          	bgez	a5,109a4 <__divsf3+0x2b8>
   10a58:	00000413          	li	s0,0
   10a5c:	00100713          	li	a4,1
   10a60:	f49ff06f          	j	109a8 <__divsf3+0x2bc>
   10a64:	00400437          	lui	s0,0x400
   10a68:	0ff00713          	li	a4,255
   10a6c:	00000913          	li	s2,0
   10a70:	f39ff06f          	j	109a8 <__divsf3+0x2bc>

00010a74 <__eqsf2>:
   10a74:	008007b7          	lui	a5,0x800
   10a78:	fff78793          	addi	a5,a5,-1 # 7fffff <__global_pointer$+0x7ed587>
   10a7c:	01755713          	srli	a4,a0,0x17
   10a80:	00a7f833          	and	a6,a5,a0
   10a84:	0175d613          	srli	a2,a1,0x17
   10a88:	01f55693          	srli	a3,a0,0x1f
   10a8c:	0ff77713          	andi	a4,a4,255
   10a90:	0ff00513          	li	a0,255
   10a94:	00b7f7b3          	and	a5,a5,a1
   10a98:	0ff67613          	andi	a2,a2,255
   10a9c:	01f5d593          	srli	a1,a1,0x1f
   10aa0:	00a71663          	bne	a4,a0,10aac <__eqsf2+0x38>
   10aa4:	00100513          	li	a0,1
   10aa8:	02081a63          	bnez	a6,10adc <__eqsf2+0x68>
   10aac:	0ff00513          	li	a0,255
   10ab0:	00a61663          	bne	a2,a0,10abc <__eqsf2+0x48>
   10ab4:	00100513          	li	a0,1
   10ab8:	02079263          	bnez	a5,10adc <__eqsf2+0x68>
   10abc:	00100513          	li	a0,1
   10ac0:	00c71e63          	bne	a4,a2,10adc <__eqsf2+0x68>
   10ac4:	00f81c63          	bne	a6,a5,10adc <__eqsf2+0x68>
   10ac8:	00b68863          	beq	a3,a1,10ad8 <__eqsf2+0x64>
   10acc:	00071863          	bnez	a4,10adc <__eqsf2+0x68>
   10ad0:	01003533          	snez	a0,a6
   10ad4:	00008067          	ret
   10ad8:	00000513          	li	a0,0
   10adc:	00008067          	ret

00010ae0 <__floatsisf>:
   10ae0:	ff010113          	addi	sp,sp,-16
   10ae4:	00112623          	sw	ra,12(sp)
   10ae8:	00812423          	sw	s0,8(sp)
   10aec:	00912223          	sw	s1,4(sp)
   10af0:	00050793          	mv	a5,a0
   10af4:	0e050463          	beqz	a0,10bdc <__floatsisf+0xfc>
   10af8:	41f55713          	srai	a4,a0,0x1f
   10afc:	00a74433          	xor	s0,a4,a0
   10b00:	40e40433          	sub	s0,s0,a4
   10b04:	01f55493          	srli	s1,a0,0x1f
   10b08:	00040513          	mv	a0,s0
   10b0c:	1b4000ef          	jal	ra,10cc0 <__clzsi2>
   10b10:	09e00793          	li	a5,158
   10b14:	40a787b3          	sub	a5,a5,a0
   10b18:	09600713          	li	a4,150
   10b1c:	04f74263          	blt	a4,a5,10b60 <__floatsisf+0x80>
   10b20:	00800713          	li	a4,8
   10b24:	00a75663          	bge	a4,a0,10b30 <__floatsisf+0x50>
   10b28:	ff850513          	addi	a0,a0,-8
   10b2c:	00a41433          	sll	s0,s0,a0
   10b30:	00941413          	slli	s0,s0,0x9
   10b34:	0ff7f793          	andi	a5,a5,255
   10b38:	01779793          	slli	a5,a5,0x17
   10b3c:	00945413          	srli	s0,s0,0x9
   10b40:	01f49513          	slli	a0,s1,0x1f
   10b44:	00f46433          	or	s0,s0,a5
   10b48:	00a46533          	or	a0,s0,a0
   10b4c:	00c12083          	lw	ra,12(sp)
   10b50:	00812403          	lw	s0,8(sp)
   10b54:	00412483          	lw	s1,4(sp)
   10b58:	01010113          	addi	sp,sp,16
   10b5c:	00008067          	ret
   10b60:	09900713          	li	a4,153
   10b64:	02f75063          	bge	a4,a5,10b84 <__floatsisf+0xa4>
   10b68:	00500713          	li	a4,5
   10b6c:	40a70733          	sub	a4,a4,a0
   10b70:	01b50693          	addi	a3,a0,27
   10b74:	00e45733          	srl	a4,s0,a4
   10b78:	00d41433          	sll	s0,s0,a3
   10b7c:	00803433          	snez	s0,s0
   10b80:	00876433          	or	s0,a4,s0
   10b84:	00500713          	li	a4,5
   10b88:	00a75663          	bge	a4,a0,10b94 <__floatsisf+0xb4>
   10b8c:	ffb50713          	addi	a4,a0,-5
   10b90:	00e41433          	sll	s0,s0,a4
   10b94:	fc000737          	lui	a4,0xfc000
   10b98:	fff70713          	addi	a4,a4,-1 # fbffffff <__global_pointer$+0xfbfed587>
   10b9c:	00747693          	andi	a3,s0,7
   10ba0:	00e47733          	and	a4,s0,a4
   10ba4:	00068a63          	beqz	a3,10bb8 <__floatsisf+0xd8>
   10ba8:	00f47413          	andi	s0,s0,15
   10bac:	00400693          	li	a3,4
   10bb0:	00d40463          	beq	s0,a3,10bb8 <__floatsisf+0xd8>
   10bb4:	00470713          	addi	a4,a4,4
   10bb8:	00571693          	slli	a3,a4,0x5
   10bbc:	0006dc63          	bgez	a3,10bd4 <__floatsisf+0xf4>
   10bc0:	fc0007b7          	lui	a5,0xfc000
   10bc4:	fff78793          	addi	a5,a5,-1 # fbffffff <__global_pointer$+0xfbfed587>
   10bc8:	00f77733          	and	a4,a4,a5
   10bcc:	09f00793          	li	a5,159
   10bd0:	40a787b3          	sub	a5,a5,a0
   10bd4:	00375413          	srli	s0,a4,0x3
   10bd8:	f59ff06f          	j	10b30 <__floatsisf+0x50>
   10bdc:	00000493          	li	s1,0
   10be0:	00000413          	li	s0,0
   10be4:	f4dff06f          	j	10b30 <__floatsisf+0x50>

00010be8 <__mulsi3>:
   10be8:	00050613          	mv	a2,a0
   10bec:	00000513          	li	a0,0
   10bf0:	0015f693          	andi	a3,a1,1
   10bf4:	00068463          	beqz	a3,10bfc <__mulsi3+0x14>
   10bf8:	00c50533          	add	a0,a0,a2
   10bfc:	0015d593          	srli	a1,a1,0x1
   10c00:	00161613          	slli	a2,a2,0x1
   10c04:	fe0596e3          	bnez	a1,10bf0 <__mulsi3+0x8>
   10c08:	00008067          	ret

00010c0c <__divsi3>:
   10c0c:	06054063          	bltz	a0,10c6c <__umodsi3+0x10>
   10c10:	0605c663          	bltz	a1,10c7c <__umodsi3+0x20>

00010c14 <__udivsi3>:
   10c14:	00058613          	mv	a2,a1
   10c18:	00050593          	mv	a1,a0
   10c1c:	fff00513          	li	a0,-1
   10c20:	02060c63          	beqz	a2,10c58 <__udivsi3+0x44>
   10c24:	00100693          	li	a3,1
   10c28:	00b67a63          	bgeu	a2,a1,10c3c <__udivsi3+0x28>
   10c2c:	00c05863          	blez	a2,10c3c <__udivsi3+0x28>
   10c30:	00161613          	slli	a2,a2,0x1
   10c34:	00169693          	slli	a3,a3,0x1
   10c38:	feb66ae3          	bltu	a2,a1,10c2c <__udivsi3+0x18>
   10c3c:	00000513          	li	a0,0
   10c40:	00c5e663          	bltu	a1,a2,10c4c <__udivsi3+0x38>
   10c44:	40c585b3          	sub	a1,a1,a2
   10c48:	00d56533          	or	a0,a0,a3
   10c4c:	0016d693          	srli	a3,a3,0x1
   10c50:	00165613          	srli	a2,a2,0x1
   10c54:	fe0696e3          	bnez	a3,10c40 <__udivsi3+0x2c>
   10c58:	00008067          	ret

00010c5c <__umodsi3>:
   10c5c:	00008293          	mv	t0,ra
   10c60:	fb5ff0ef          	jal	ra,10c14 <__udivsi3>
   10c64:	00058513          	mv	a0,a1
   10c68:	00028067          	jr	t0
   10c6c:	40a00533          	neg	a0,a0
   10c70:	0005d863          	bgez	a1,10c80 <__umodsi3+0x24>
   10c74:	40b005b3          	neg	a1,a1
   10c78:	f9dff06f          	j	10c14 <__udivsi3>
   10c7c:	40b005b3          	neg	a1,a1
   10c80:	00008293          	mv	t0,ra
   10c84:	f91ff0ef          	jal	ra,10c14 <__udivsi3>
   10c88:	40a00533          	neg	a0,a0
   10c8c:	00028067          	jr	t0

00010c90 <__modsi3>:
   10c90:	00008293          	mv	t0,ra
   10c94:	0005ca63          	bltz	a1,10ca8 <__modsi3+0x18>
   10c98:	00054c63          	bltz	a0,10cb0 <__modsi3+0x20>
   10c9c:	f79ff0ef          	jal	ra,10c14 <__udivsi3>
   10ca0:	00058513          	mv	a0,a1
   10ca4:	00028067          	jr	t0
   10ca8:	40b005b3          	neg	a1,a1
   10cac:	fe0558e3          	bgez	a0,10c9c <__modsi3+0xc>
   10cb0:	40a00533          	neg	a0,a0
   10cb4:	f61ff0ef          	jal	ra,10c14 <__udivsi3>
   10cb8:	40b00533          	neg	a0,a1
   10cbc:	00028067          	jr	t0

00010cc0 <__clzsi2>:
   10cc0:	000107b7          	lui	a5,0x10
   10cc4:	02f57a63          	bgeu	a0,a5,10cf8 <__clzsi2+0x38>
   10cc8:	0ff00793          	li	a5,255
   10ccc:	00a7b7b3          	sltu	a5,a5,a0
   10cd0:	00379793          	slli	a5,a5,0x3
   10cd4:	02000713          	li	a4,32
   10cd8:	40f70733          	sub	a4,a4,a5
   10cdc:	00f557b3          	srl	a5,a0,a5
   10ce0:	00000517          	auipc	a0,0x0
   10ce4:	48850513          	addi	a0,a0,1160 # 11168 <__clz_tab>
   10ce8:	00f507b3          	add	a5,a0,a5
   10cec:	0007c503          	lbu	a0,0(a5) # 10000 <register_fini-0x74>
   10cf0:	40a70533          	sub	a0,a4,a0
   10cf4:	00008067          	ret
   10cf8:	01000737          	lui	a4,0x1000
   10cfc:	01000793          	li	a5,16
   10d00:	fce56ae3          	bltu	a0,a4,10cd4 <__clzsi2+0x14>
   10d04:	01800793          	li	a5,24
   10d08:	fcdff06f          	j	10cd4 <__clzsi2+0x14>

00010d0c <atexit>:
   10d0c:	00050593          	mv	a1,a0
   10d10:	00000693          	li	a3,0
   10d14:	00000613          	li	a2,0
   10d18:	00000513          	li	a0,0
   10d1c:	2080006f          	j	10f24 <__register_exitproc>

00010d20 <exit>:
   10d20:	ff010113          	addi	sp,sp,-16
   10d24:	00000593          	li	a1,0
   10d28:	00812423          	sw	s0,8(sp)
   10d2c:	00112623          	sw	ra,12(sp)
   10d30:	00050413          	mv	s0,a0
   10d34:	28c000ef          	jal	ra,10fc0 <__call_exitprocs>
   10d38:	c3418793          	addi	a5,gp,-972 # 126ac <_global_impure_ptr>
   10d3c:	0007a503          	lw	a0,0(a5)
   10d40:	03c52783          	lw	a5,60(a0)
   10d44:	00078463          	beqz	a5,10d4c <exit+0x2c>
   10d48:	000780e7          	jalr	a5
   10d4c:	00040513          	mv	a0,s0
   10d50:	38c000ef          	jal	ra,110dc <_exit>

00010d54 <__libc_fini_array>:
   10d54:	ff010113          	addi	sp,sp,-16
   10d58:	00812423          	sw	s0,8(sp)
   10d5c:	00001797          	auipc	a5,0x1
   10d60:	51878793          	addi	a5,a5,1304 # 12274 <__init_array_end>
   10d64:	00001417          	auipc	s0,0x1
   10d68:	51440413          	addi	s0,s0,1300 # 12278 <__fini_array_end>
   10d6c:	40f40433          	sub	s0,s0,a5
   10d70:	00112623          	sw	ra,12(sp)
   10d74:	00912223          	sw	s1,4(sp)
   10d78:	40245413          	srai	s0,s0,0x2
   10d7c:	02040263          	beqz	s0,10da0 <__libc_fini_array+0x4c>
   10d80:	00241493          	slli	s1,s0,0x2
   10d84:	ffc48493          	addi	s1,s1,-4
   10d88:	00f484b3          	add	s1,s1,a5
   10d8c:	0004a783          	lw	a5,0(s1)
   10d90:	fff40413          	addi	s0,s0,-1
   10d94:	ffc48493          	addi	s1,s1,-4
   10d98:	000780e7          	jalr	a5
   10d9c:	fe0418e3          	bnez	s0,10d8c <__libc_fini_array+0x38>
   10da0:	00c12083          	lw	ra,12(sp)
   10da4:	00812403          	lw	s0,8(sp)
   10da8:	00412483          	lw	s1,4(sp)
   10dac:	01010113          	addi	sp,sp,16
   10db0:	00008067          	ret

00010db4 <__libc_init_array>:
   10db4:	ff010113          	addi	sp,sp,-16
   10db8:	00812423          	sw	s0,8(sp)
   10dbc:	01212023          	sw	s2,0(sp)
   10dc0:	00001417          	auipc	s0,0x1
   10dc4:	4ac40413          	addi	s0,s0,1196 # 1226c <__init_array_start>
   10dc8:	00001917          	auipc	s2,0x1
   10dcc:	4a490913          	addi	s2,s2,1188 # 1226c <__init_array_start>
   10dd0:	40890933          	sub	s2,s2,s0
   10dd4:	00112623          	sw	ra,12(sp)
   10dd8:	00912223          	sw	s1,4(sp)
   10ddc:	40295913          	srai	s2,s2,0x2
   10de0:	00090e63          	beqz	s2,10dfc <__libc_init_array+0x48>
   10de4:	00000493          	li	s1,0
   10de8:	00042783          	lw	a5,0(s0)
   10dec:	00148493          	addi	s1,s1,1
   10df0:	00440413          	addi	s0,s0,4
   10df4:	000780e7          	jalr	a5
   10df8:	fe9918e3          	bne	s2,s1,10de8 <__libc_init_array+0x34>
   10dfc:	00001417          	auipc	s0,0x1
   10e00:	47040413          	addi	s0,s0,1136 # 1226c <__init_array_start>
   10e04:	00001917          	auipc	s2,0x1
   10e08:	47090913          	addi	s2,s2,1136 # 12274 <__init_array_end>
   10e0c:	40890933          	sub	s2,s2,s0
   10e10:	40295913          	srai	s2,s2,0x2
   10e14:	00090e63          	beqz	s2,10e30 <__libc_init_array+0x7c>
   10e18:	00000493          	li	s1,0
   10e1c:	00042783          	lw	a5,0(s0)
   10e20:	00148493          	addi	s1,s1,1
   10e24:	00440413          	addi	s0,s0,4
   10e28:	000780e7          	jalr	a5
   10e2c:	fe9918e3          	bne	s2,s1,10e1c <__libc_init_array+0x68>
   10e30:	00c12083          	lw	ra,12(sp)
   10e34:	00812403          	lw	s0,8(sp)
   10e38:	00412483          	lw	s1,4(sp)
   10e3c:	00012903          	lw	s2,0(sp)
   10e40:	01010113          	addi	sp,sp,16
   10e44:	00008067          	ret

00010e48 <memset>:
   10e48:	00f00313          	li	t1,15
   10e4c:	00050713          	mv	a4,a0
   10e50:	02c37e63          	bgeu	t1,a2,10e8c <memset+0x44>
   10e54:	00f77793          	andi	a5,a4,15
   10e58:	0a079063          	bnez	a5,10ef8 <memset+0xb0>
   10e5c:	08059263          	bnez	a1,10ee0 <memset+0x98>
   10e60:	ff067693          	andi	a3,a2,-16
   10e64:	00f67613          	andi	a2,a2,15
   10e68:	00e686b3          	add	a3,a3,a4
   10e6c:	00b72023          	sw	a1,0(a4) # 1000000 <__global_pointer$+0xfed588>
   10e70:	00b72223          	sw	a1,4(a4)
   10e74:	00b72423          	sw	a1,8(a4)
   10e78:	00b72623          	sw	a1,12(a4)
   10e7c:	01070713          	addi	a4,a4,16
   10e80:	fed766e3          	bltu	a4,a3,10e6c <memset+0x24>
   10e84:	00061463          	bnez	a2,10e8c <memset+0x44>
   10e88:	00008067          	ret
   10e8c:	40c306b3          	sub	a3,t1,a2
   10e90:	00269693          	slli	a3,a3,0x2
   10e94:	00000297          	auipc	t0,0x0
   10e98:	005686b3          	add	a3,a3,t0
   10e9c:	00c68067          	jr	12(a3)
   10ea0:	00b70723          	sb	a1,14(a4)
   10ea4:	00b706a3          	sb	a1,13(a4)
   10ea8:	00b70623          	sb	a1,12(a4)
   10eac:	00b705a3          	sb	a1,11(a4)
   10eb0:	00b70523          	sb	a1,10(a4)
   10eb4:	00b704a3          	sb	a1,9(a4)
   10eb8:	00b70423          	sb	a1,8(a4)
   10ebc:	00b703a3          	sb	a1,7(a4)
   10ec0:	00b70323          	sb	a1,6(a4)
   10ec4:	00b702a3          	sb	a1,5(a4)
   10ec8:	00b70223          	sb	a1,4(a4)
   10ecc:	00b701a3          	sb	a1,3(a4)
   10ed0:	00b70123          	sb	a1,2(a4)
   10ed4:	00b700a3          	sb	a1,1(a4)
   10ed8:	00b70023          	sb	a1,0(a4)
   10edc:	00008067          	ret
   10ee0:	0ff5f593          	andi	a1,a1,255
   10ee4:	00859693          	slli	a3,a1,0x8
   10ee8:	00d5e5b3          	or	a1,a1,a3
   10eec:	01059693          	slli	a3,a1,0x10
   10ef0:	00d5e5b3          	or	a1,a1,a3
   10ef4:	f6dff06f          	j	10e60 <memset+0x18>
   10ef8:	00279693          	slli	a3,a5,0x2
   10efc:	00000297          	auipc	t0,0x0
   10f00:	005686b3          	add	a3,a3,t0
   10f04:	00008293          	mv	t0,ra
   10f08:	fa0680e7          	jalr	-96(a3)
   10f0c:	00028093          	mv	ra,t0
   10f10:	ff078793          	addi	a5,a5,-16
   10f14:	40f70733          	sub	a4,a4,a5
   10f18:	00f60633          	add	a2,a2,a5
   10f1c:	f6c378e3          	bgeu	t1,a2,10e8c <memset+0x44>
   10f20:	f3dff06f          	j	10e5c <memset+0x14>

00010f24 <__register_exitproc>:
   10f24:	c3418793          	addi	a5,gp,-972 # 126ac <_global_impure_ptr>
   10f28:	0007a703          	lw	a4,0(a5)
   10f2c:	14872783          	lw	a5,328(a4)
   10f30:	04078c63          	beqz	a5,10f88 <__register_exitproc+0x64>
   10f34:	0047a703          	lw	a4,4(a5)
   10f38:	01f00813          	li	a6,31
   10f3c:	06e84e63          	blt	a6,a4,10fb8 <__register_exitproc+0x94>
   10f40:	00271813          	slli	a6,a4,0x2
   10f44:	02050663          	beqz	a0,10f70 <__register_exitproc+0x4c>
   10f48:	01078333          	add	t1,a5,a6
   10f4c:	08c32423          	sw	a2,136(t1) # 101c0 <bottle_status+0x7c>
   10f50:	1887a883          	lw	a7,392(a5)
   10f54:	00100613          	li	a2,1
   10f58:	00e61633          	sll	a2,a2,a4
   10f5c:	00c8e8b3          	or	a7,a7,a2
   10f60:	1917a423          	sw	a7,392(a5)
   10f64:	10d32423          	sw	a3,264(t1)
   10f68:	00200693          	li	a3,2
   10f6c:	02d50463          	beq	a0,a3,10f94 <__register_exitproc+0x70>
   10f70:	00170713          	addi	a4,a4,1
   10f74:	00e7a223          	sw	a4,4(a5)
   10f78:	010787b3          	add	a5,a5,a6
   10f7c:	00b7a423          	sw	a1,8(a5)
   10f80:	00000513          	li	a0,0
   10f84:	00008067          	ret
   10f88:	14c70793          	addi	a5,a4,332
   10f8c:	14f72423          	sw	a5,328(a4)
   10f90:	fa5ff06f          	j	10f34 <__register_exitproc+0x10>
   10f94:	18c7a683          	lw	a3,396(a5)
   10f98:	00170713          	addi	a4,a4,1
   10f9c:	00e7a223          	sw	a4,4(a5)
   10fa0:	00c6e633          	or	a2,a3,a2
   10fa4:	18c7a623          	sw	a2,396(a5)
   10fa8:	010787b3          	add	a5,a5,a6
   10fac:	00b7a423          	sw	a1,8(a5)
   10fb0:	00000513          	li	a0,0
   10fb4:	00008067          	ret
   10fb8:	fff00513          	li	a0,-1
   10fbc:	00008067          	ret

00010fc0 <__call_exitprocs>:
   10fc0:	fd010113          	addi	sp,sp,-48
   10fc4:	c3418793          	addi	a5,gp,-972 # 126ac <_global_impure_ptr>
   10fc8:	01812423          	sw	s8,8(sp)
   10fcc:	0007ac03          	lw	s8,0(a5)
   10fd0:	01312e23          	sw	s3,28(sp)
   10fd4:	01412c23          	sw	s4,24(sp)
   10fd8:	01512a23          	sw	s5,20(sp)
   10fdc:	01612823          	sw	s6,16(sp)
   10fe0:	02112623          	sw	ra,44(sp)
   10fe4:	02812423          	sw	s0,40(sp)
   10fe8:	02912223          	sw	s1,36(sp)
   10fec:	03212023          	sw	s2,32(sp)
   10ff0:	01712623          	sw	s7,12(sp)
   10ff4:	00050a93          	mv	s5,a0
   10ff8:	00058b13          	mv	s6,a1
   10ffc:	00100a13          	li	s4,1
   11000:	fff00993          	li	s3,-1
   11004:	148c2903          	lw	s2,328(s8)
   11008:	02090863          	beqz	s2,11038 <__call_exitprocs+0x78>
   1100c:	00492483          	lw	s1,4(s2)
   11010:	fff48413          	addi	s0,s1,-1
   11014:	02044263          	bltz	s0,11038 <__call_exitprocs+0x78>
   11018:	00249493          	slli	s1,s1,0x2
   1101c:	009904b3          	add	s1,s2,s1
   11020:	040b0463          	beqz	s6,11068 <__call_exitprocs+0xa8>
   11024:	1044a783          	lw	a5,260(s1)
   11028:	05678063          	beq	a5,s6,11068 <__call_exitprocs+0xa8>
   1102c:	fff40413          	addi	s0,s0,-1
   11030:	ffc48493          	addi	s1,s1,-4
   11034:	ff3416e3          	bne	s0,s3,11020 <__call_exitprocs+0x60>
   11038:	02c12083          	lw	ra,44(sp)
   1103c:	02812403          	lw	s0,40(sp)
   11040:	02412483          	lw	s1,36(sp)
   11044:	02012903          	lw	s2,32(sp)
   11048:	01c12983          	lw	s3,28(sp)
   1104c:	01812a03          	lw	s4,24(sp)
   11050:	01412a83          	lw	s5,20(sp)
   11054:	01012b03          	lw	s6,16(sp)
   11058:	00c12b83          	lw	s7,12(sp)
   1105c:	00812c03          	lw	s8,8(sp)
   11060:	03010113          	addi	sp,sp,48
   11064:	00008067          	ret
   11068:	00492783          	lw	a5,4(s2)
   1106c:	0044a683          	lw	a3,4(s1)
   11070:	fff78793          	addi	a5,a5,-1
   11074:	04878a63          	beq	a5,s0,110c8 <__call_exitprocs+0x108>
   11078:	0004a223          	sw	zero,4(s1)
   1107c:	fa0688e3          	beqz	a3,1102c <__call_exitprocs+0x6c>
   11080:	18892783          	lw	a5,392(s2)
   11084:	008a1733          	sll	a4,s4,s0
   11088:	00492b83          	lw	s7,4(s2)
   1108c:	00f777b3          	and	a5,a4,a5
   11090:	00079e63          	bnez	a5,110ac <__call_exitprocs+0xec>
   11094:	000680e7          	jalr	a3
   11098:	00492783          	lw	a5,4(s2)
   1109c:	f77794e3          	bne	a5,s7,11004 <__call_exitprocs+0x44>
   110a0:	148c2783          	lw	a5,328(s8)
   110a4:	f92784e3          	beq	a5,s2,1102c <__call_exitprocs+0x6c>
   110a8:	f5dff06f          	j	11004 <__call_exitprocs+0x44>
   110ac:	18c92783          	lw	a5,396(s2)
   110b0:	0844a583          	lw	a1,132(s1)
   110b4:	00f77733          	and	a4,a4,a5
   110b8:	00071c63          	bnez	a4,110d0 <__call_exitprocs+0x110>
   110bc:	000a8513          	mv	a0,s5
   110c0:	000680e7          	jalr	a3
   110c4:	fd5ff06f          	j	11098 <__call_exitprocs+0xd8>
   110c8:	00892223          	sw	s0,4(s2)
   110cc:	fb1ff06f          	j	1107c <__call_exitprocs+0xbc>
   110d0:	00058513          	mv	a0,a1
   110d4:	000680e7          	jalr	a3
   110d8:	fc1ff06f          	j	11098 <__call_exitprocs+0xd8>

000110dc <_exit>:
   110dc:	00000593          	li	a1,0
   110e0:	00000613          	li	a2,0
   110e4:	00000693          	li	a3,0
   110e8:	00000713          	li	a4,0
   110ec:	00000793          	li	a5,0
   110f0:	05d00893          	li	a7,93
   110f4:	00000073          	ecall
   110f8:	00054463          	bltz	a0,11100 <_exit+0x24>
   110fc:	0000006f          	j	110fc <_exit+0x20>
   11100:	ff010113          	addi	sp,sp,-16
   11104:	00812423          	sw	s0,8(sp)
   11108:	00050413          	mv	s0,a0
   1110c:	00112623          	sw	ra,12(sp)
   11110:	40800433          	neg	s0,s0
   11114:	00c000ef          	jal	ra,11120 <__errno>
   11118:	00852023          	sw	s0,0(a0)
   1111c:	0000006f          	j	1111c <_exit+0x40>

00011120 <__errno>:
   11120:	c3c18793          	addi	a5,gp,-964 # 126b4 <_impure_ptr>
   11124:	0007a503          	lw	a0,0(a5)
   11128:	00008067          	ret
```

## Unique Instructions
```
Number of different instructions: 40
List of unique instructions:
jr
sltu
sw
sb
add
blt
bge
lw
ecall
xor
bltu
srli
or
beqz
blez
lui
andi
bne
auipc
bgeu
mv
bgez
li
neg
srl
beq
jal
j
bnez
jalr
lbu
addi
slli
srai
snez
sll
and
sub
bltz
ret
```
## Word of thanks
I sciencerly thank Mr. Kunal Gosh(Founder/VSD) for helping me out to complete this flow smoothly.

## Acknowledgement
1. Kunal Ghosh, VSD Corp. Pvt. Ltd.
2. Skywater Foundry
3. Alwin Shaju,Colleague,IIIT B
4. Mayank Kabra

## Reference
1. https://github.com/SakethGajawada/RISCV-GNU
2. https://how2electronics.com/arduino-water-flow-sensor-measure-flow-rate-volume/
