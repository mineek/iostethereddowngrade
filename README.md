Tethered Downgrade Guide
By Mineek

discord: Mineek#6323

This tutorial was made in half an hour, its really bad but should get you started on your tethered downgrade adventure!

Note: A10+ Devices DONT have kpp! (YOU CAN STILL DOWNGRADE, JUST SKIP THE KPP PARTS!
like instead of: 

	pyimg4 im4p extract -i kernelcache -o kcache.raw --extra kpp.bin

you do: 

	pyimg4 im4p extract -i kernelcache -o kcache.raw

HUGE THANKS TO **galaxy#6181**. Without him, I wouldn't have known all this to write a guide!

# REQUIREMENTS:
- irecovery
- futurerestore
- pyimg4 (pip3 install pyimg4) (**MAKE SURE YOU UPDATED PYTHON AND NOT USING THE BUNDLED ONE!**)
- iBoot64patcher (https://github.com/Cryptiiiic/iBoot64Patcher)
- Kernel64patcher (https://github.com/iSuns9/Kernel64Patcher)
- img4tool (https://github.com/tihmstar/img4tool)
- img4 (https://github.com/xerub/img4lib)
- ldid (https://github.com/ProcursusTeam/ldid)
- restored_external64_patcher (https://github.com/iSuns9/restored_external64patcher)
- asr64_patcher (https://github.com/exploit3dguy/asr64_patcher)

**Make sure to use the forks listed above.**

# Downgrade portion:

1. Grab yourself your ipsw for iOS 14.3
2. Extract it and grab yourself your kernel cache and restore_ramdisk
3. Extract the restore_ramdisk with: img4 -i restore_ramdisk -o ramdisk.dmg
4. Mount it: 


		mkdir ramdisk && hdiutil attach ramdisk.dmg -mountpoint ramdisk


5. patch the ASR in the ramdisk: 


		asr64_patcher ramdisk/usr/sbin/asr patched_asr


6. resign it:


		ldid -e ramdisk/usr/sbin/asr > ents.plist
		ldid -Sents.plist patched_asr


7. Grab your restored_external: 


		cp ramdisk/usr/local/bin/restored_external .


8. Patch it: 


		restored_external64_patcher restored_external restored_external_patched


9. Extract the ents: 


		ldid -e restored_external > restored_externel_ents.plist


10. Remove the old ones: 


		rm ramdisk/usr/sbin/asr && rm ramdisk/usr/local/bin/restored_external


11. Resign it: 


		ldid -Srestored_externel_ents.plist restored_external_patched


12. chmod them: 


		chmod -R 755 restored_external_patched
		chmod -R 755 patched_asr


13. Copy them back: 


		cp -a restored_external_patched ramdisk/usr/local/bin/restored_external
		cp -a patched_asr ramdisk/usr/sbin/asr


14. Detach from the ramdisk: 


		hdiutil detach ramdisk


15. Rebuild the ramdisk (dont sign it tho, futurerestore will):


		pyimg4 im4p create -i ramdisk.dmg -o ramdisk.im4p -f rdsk


16. Extract the kernel:
	

		pyimg4 im4p extract -i kernelcache -o kcache.raw --extra kpp.bin 


(leave out --extra kpp.bin if you dont have kpp)

17. Patch it: 


		Kernel64Patcher kcache.raw krnl.patched -f -a


18. Rebuild the kernel:


		pyimg4 im4p create -i krnl.patched -o krnl.im4p --extra kpp.bin -f rkrn --lzss (leave out --extra kpp.bin if you dont have kpp)


19. You can now restore with futurerestore via this command (blob can be for ANY version):

**(MAKE SURE YOU ARE IN PWNDFU WITH SIGCHECKS REMOVED!)**


		futurerestore -t shsh.shsh2 --use-pwndfu --skip-blob --rdsk ramdisk.im4p --rkrn krnl.im4p --latest-sep --latest-baseband ipsw.ipsw

Boot portion:

1. Prepare your ibss, ibec, devicetree, rootfs_trustcache and kernelcache.

2. Prepare your iv keys for ibss and ibec.

3. decrypt ibss and ibec:


		img4 -i ibss -o ibss.dmg -k ibss_ivkey
		img4 -i ibec -o ibec.dmg -k ibec_ivkey


4. Patch them:


		iBoot64Patcher ibss.dmg ibss.patched
		iBoot64Patcher ibec.dmg ibec.patched -b "-v"


5. Repack them with your IM4M (you can get it by doing this: img4tool -e -s yourshsh.shsh2 -m IM4M)


		img4 -i ibss.patched -o ibss.img4 -M IM4M -A -T ibss
		img4 -i ibec.patched -o ibec.img4 -M IM4M -A -T ibec


6. Sign your devicetree and rootfs_trustcache: (and also the firmware files in the ipsw)


		img4 -i devicetree -o devicetree.img4 -M IM4M -T rdtr
		img4 -i rootfs_trustcache -o rootfs_trustcache.img4 -M IM4M -T rtsc


7. Extract the kernelcache:


		pyimg4 im4p extract -i kernelcache -o kcache.raw --extra kpp.bin 

(leave out --extra kpp.bin if you dont have kpp)

8. Patch it 
* the reason we don't use amfi patches is because jailbreak doesnt work anymore if you use amfi patches. 
* Make sure to DO amfi patches when restoring tho. 


		Kernel64Patcher kcache.raw krnlboot.patched -f


9. Repack it:


		pyimg4 im4p create -i krnlboot.patched -o krnlboot.im4p --extra kpp.bin -f rkrn --lzss
		pyimg4 img4 create -p krnlboot.im4p -o krnlboot.img4 -m IM4M


10. Boot: 
**(MAKE SURE YOU USE IPWNDFU TO ACTIVATE, IF YOU USE GASTER YOU CANNOT ACTIVATE THE DEVICE!)**


		irecovery -f iBSS.img4
		irecovery -f iBEC.img4

  
  If you have a10 or higher use this:
  irecovery -c go


	irecovery -f devicetree.img4
	irecovery -c devicetree
	# if you have firmware add them here like this:
	# MAKE SURE TO SIGN THEM!
	# irecovery -f yourfirmware.img4
	# irecovery -c firmware
	irecovery -f aop.img4
	irecovery -c firmware
	irecovery -f rootfs_trustcache.img4
	irecovery -c firmware
	irecovery -f krnlboot.img4
	irecovery -c bootx
