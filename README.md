# LAB_Svalbard
#!/usr/bin/env bash
#		Topography & LAB of Svalbard + profile through Bockfjorden
gmt begin Svalbard_crossection_vs
	#creating mean surface for LAB and extracting cross section from 3D datasets
	gmt set MAP_ANNOT_OBLIQUE separate
         # gmt blockmean LAB_svalbard_new.xyz -R-44/112/62/90 -I0.5 > mean.xyz
	# gmt surface mean.xyz -I0.1 -Gdata.nc
	#Setting coordinates for cross section
	gmt project -C20/79.4 -E-0/79.4 -G1 -N > track
	gmt grdtrack track -Gbath1.nc > Topo
	gmt grdtrack track -Gdata.nc  > LAB
	gmt grdtrack track -GArcCRUST-Moho-to67.nc  > Moho
	gmt grdtrack track -GArcCRUST-SedimentThickness-to67.nc  > Sed_thickness

	#Creating contours of Vs for cross section
	gmt grdinterpolate Barents50.nc -E0/79.4/20/79.4+i.1d+g+p -T0/150/1 -Gb50slice.nc
	gmt grdedit b50slice.nc -R0/20/0/150000 -A -Gb50m.nc
	gmt grdclip -Sa4.75/4.75 -Sb1.0/1.0 b50m.nc -Gb50m_clip.nc
	gmt makecpt -Cjet -T1,2,3,3.5,3.7,4,4.2,4.5,4.7,4.8 -N
	gmt grdimage -R0/20/-5000/120000 -JX35c/-15c b50m_clip.nc 
	gmt grdcontour b50m_clip.nc -A1,2,3,3.5,3.7,4,4.2,4.5,4.7,4.8 -C1,2,3,3.5,3.7,4,4.2,4.5,4.7,4.8 -W0.5p,black,.

	#Plotting LAB, Topography, Top basement and Moho on cross section. Data adjustments converting to km and change +/- numbers. Plotting title for graph.
	gmt plot -R0/20/-5000/120000 -JX35c/-15c LAB -W1.5p,. -Bx1 -By5000 -i0,3 -B+t"S-wave velocity profile through study area"
	awk '{print $1, -$4}' Topo | gmt plot -W1p,-
	gmt math Topo Sed_thickness SUB = Top_basement
	awk '{print $1, -$4}' Top_basement | gmt plot -W1p,.
	gmt plot Moho -W1.5p,- -i0,3 -By+l"DEPTH (M)" -Bx+l"LONGITUDE (@~\260@~E)"
	gmt plot Cordinates_Bockfjorden.txt -Sa0.7c -Gred -i0,2
	gmt plot Xenoliths_HAM_2.txt -Sd0.3c -Gwhite -i4,3
	gmt plot Xenoliths_BVC23.txt -Sd0.3c -Ggreen -i4,3

	#plotting legend for cross section
	gmt legend -DjLB+w6.5c+o0.25c -F+p1p+gbeige+s <<- EOF
	S 0.5c - 0.9c - 1.5p,. 1c LAB
	S 0.5c - 0.9c - 1.5p,- 1c Moho
	S 0.5c - 0.9c - 0.5p,- 1c Topography
	S 0.5c - 0.9c - 0.5p,. 1c Top Basement
	S 0.5c d 0.3c green - 1c BVC23X samples
	S 0.5c d 0.3c white - 1c Samples from Amundsen(1978)
	EOF

	#Plotting 2 maps showing topography and LAB surface
	gmt subplot begin 1x2 -A+JTL -Fs16c/9c -M1 -R-100/-1600/650/-550r+uk -Js0/90/1:5000000 -B1 -T"Svalbard Topo and LAB" -Yh+1c
		#Plotting topography of area
		gmt subplot set 0,0 -Ce3c 
		gmt grd2cpt bath1.nc -Crelief -Z
		gmt grdimage bath1.nc -I+a0 -B+t"Topography"
		gmt colorbar -DJRM+o1c/0+e+mc -Bx500+lTOPO -By+lkm
		gmt pscoast -Dh -W0.75p,black
		gmt plot track -W2p,0/0/0
		gmt plot Cordinates_Bockfjorden.txt -Sa0.7c -Gred
		awk '{print $1, $2, $4}' Cordinates_Bockfjorden.txt | gmt text -D7p/0 -F+f14p,red+jTL -N

		#Plotting dept of LAB surface in area
		gmt subplot set 0,1 -Ce3c 
		gmt makecpt -Crainbow -T20000/100000/5000
		gmt grdimage data.nc -B+t"LAB depth"
		gmt colorbar -DJRM+o1c/0+e+mc -Bx5000+lLAB -By+lm
		gmt pscoast -Dh -W0.75p,black
		gmt plot track -W2p,0/0/0
		gmt plot Cordinates_Bockfjorden.txt -Sa0.7c -Gred
		awk '{print $1, $2, $4}' Cordinates_Bockfjorden.txt | gmt text -D7p/0 -F+f14p,red+jTL -N
	gmt subplot end
gmt end show
