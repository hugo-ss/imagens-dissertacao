
import os
os.environ['DATAPATH'] = './data/'

from rsf.proj import *

## Graph

def grey(title):
    return '''
    grey title="%s" 
    ''' % title
    
def graph(title):
    return '''
    graph title="%s" 
    ''' % title

def wiggle(title):
    return '''
    wiggle title="%s" 
    ''' % title

########################################################
#################   IMPORTING DATA   ###################
########################################################

## Import FOFF (Data)

Flow('cmp0','cmp.asc',
      '''
      echo in=$SOURCE n1=625 n2=125 d1=0.8 d2=0.1 o1=0 o2=0
      label1=Time unit1=ns label2=Distance unit2=m data_format=ascii_float
      ''')
Flow('cmp','cmp0','dd form=native')

Plot('cmp',grey('CMP Test')) 
# 
## Import Line to interpolate (Data)
 
npts = 625

## File 2D

Flow('line tlines','out.su',
     'suread suxdr=y tfile=${TARGETS[1]}')
     
## File 3D     

Flow('lines','line',
	 '''
	 intbin |
	 put
     label3=Source d3=0.1 o3=0  unit3=m
     label2=Offset d2=0.3 o2=0 unit2=m
     label1=Time d1=0.8 o1=0 unit1=ns
     ''')

Plot('lines',
       '''
       transp memsize=1000 plane=23 |
       byte gainpanel=each |
       grey3 frame1=1 frame2=9 frame3=50 flat=n
       title="Raw Data"
       ''')

##########################################################
##################### STEP TWO (KICK) ####################
##########################################################

## Mask for zero traces
Flow('mask','cmp',
     '''
     mul $SOURCE | stack axis=1 | 
     mask min=0.01 | dd type=float | 
     spray axis=1 n=%d d=0.8
     ''' % (npts))
Plot('mask',grey('Mask'))
# 
## Adaptive PEFs
Flow('apef mask2','cmp mask',
     '''
     apef maskin=${SOURCES[1]} maskout=${TARGETS[1]} jump=1
     a=5,7 niter=100 rect1=5 rect2=5 verb=y
     ''')

## Display PEF coefficients
Flow('fcoe','apef','window n1=1 f1=3 n2=1')
Plot('fcoe',grey('First Coefficient') + ' mean=y color=j scalebar=y')
Flow('mcoe','apef',
     'cut n1=8 n2=1 | stack axis=2 norm=n | stack axis=1 norm=n')
Plot('mcoe',grey('Mean Coefficient') + ' mean=y color=j scalebar=y')
Result('coe','fcoe mcoe','SideBySideAniso')

##########################################################
############### STEP THREE (INTERPOLATION) ###############
##########################################################

Flow('shot1','lines', 'window n2=1 f2=100')
Plot('shot1',grey('shot1 Test')) 

## Mask for zero traces
Flow('mask0','shot1',
     '''
     mul $SOURCE | stack axis=1 | 
     mask min=0.01 | dd type=float | 
     spray axis=1 n=%d d=0.8
     ''' % (npts))
Plot('mask0',grey('Mask'))

## Missing data interpolation (RNA)
Flow('amiss','shot1 apef mask0',
     'miss4 filt=${SOURCES[1]} mask=${SOURCES[2]} niter=100 exact=n verb=y')

Plot('amiss',grey('RNA Interpolation' ))

Plot('compare','cmp shot1 amiss','SideBySideAniso')

#~ ####################### SEMBLANCE #######################

#~ v0 = 0.04
#~ dv = 0.001
#~ nv = 151

#~ Flow('scn1','cmp',
        #~ '''
        #~ vscan avosemblance=y type=avo v0=%g nv=%d dv=%g half=n nb=3
        #~ ''' % (v0,nv,dv))
#~ Plot('scn1',  
        #~ '''
        #~ grey color=j allpos=y title="scn1" pclip=100
        #~ ''')

#~ Flow('vel1','scn1',
        #~ '''
        #~ scale axis=2 | pick an=2 gate=4 smooth=y rect1=20 | window
        #~ ''')

#~ Flow('vel0','scn1',
        #~ '''
        #~ scale axis=2 | pick an=2 gate=4 smooth=y rect1=20 | window
        #~ ''')

#~ Plot('vel1',  
        #~ '''
        #~ grey color=g allpos=y title="scn1" pclip=100 scalebar=y
        #~ ''')
        
#~ Plot('vel0',  
        #~ '''
        #~ graph yreverse=y transp=y min2=0.4 max2=0.19 pad=n
        #~ plotcol=7 plotfat=7 wanttitle=n wantaxis=n
        #~ ''')

#~ Plot('over','scn1 vel0','Overlay')

#~ ####################### SECTION #######################

#~ Flow('line1','lines', 'window n3=1 f2=25')
#~ Plot('line1',grey('Line Test')) 

#~ ####################### MIGRATION #######################

#~ Flow('post','line1 vel1',
     #~ '''
     #~ stolt vel=${SOURCES[1]} extend=4
     #~ ''')

#~ Plot('post',grey('Post-stack stolt'))

####################### INTERPOL #######################
## Interpolation all CS's
shots = []
for case in range(0,282):
    inp='shot%d' % (case)
    out='sint%d' % (case)
    
    Flow(inp, 'lines', 'window n2=1 f2=%d' % (case))
    Flow(out,'shot%d apef mask0' % (case),
    'miss4 filt=${SOURCES[1]} mask=${SOURCES[2]} niter=100 verb=y')
    shots.append(out)
    
    Plot(out,'grey title="(%d)" ' % (case))
    
Flow('sintf',shots,'rcat axis=3 ${SOURCES[0:%d]}' % (case))
Flow('data2d',shots,'rcat axis=2 ${SOURCES[0:%d]}' % (case))
# Flow('sints','sintf',
    # '''
    # intbin | transp plane=23 | 
    # window f1=1 f2=1 f3=1 |
    # putassim
    # label3=Source d3=0.3 o3=0  unit3=m
    # label2=Offset d2=0.1 o2=0 unit2=m
    # label1=Time d1=0.8 o1=0 unit1=ns
    # ''')
    
## Inv 3d-2-2d
# Flow('data2d','sints',
        # '''
        # intbin inv=y
        # ''')
   
## Plotting interpolated data
Plot('sintf',
       '''
       byte gainpanel=each |
       grey3 frame1=1 frame2=9 frame3=50 flat=n
       title="Raw Data"
       ''')
       
         

#~ ############### VEL MODEL #############

#~ Flow('scn','sints',
     #~ '''
     #~ vscan avosemblance=y type=avo v0=%g nv=%d dv=%g half=n nb=3
     #~ ''' % (v0,nv,dv),split=[3,10140])
     
#~ ## Picking     
#~ Flow('vel','scn',
        #~ '''
        #~ scale axis=2 | pick an=2 gate=4 smooth=y rect1=50 | window
        #~ ''')
#~ ## Plotting
#~ Plot('vel',
       #~ '''
       #~ grey title="NMO Velocity" 
       #~ label1="Time" label2="Lateral"
       #~ color=g scalebar=y barlabel="Velocity"
       #~ ''')

#~ ###################
#~ ## Smooth Velocity
#~ ###################

#~ Flow('smt_vel','vel','smooth rect1=5 rect2=5 repeat=10')

#~ Plot('smt_vel',
       #~ '''
       #~ grey title="smt_vel" 
       #~ label1="Time" label2="Lateral"
       #~ color=g scalebar=y barlabel="Velocity"
       #~ ''')
       
#~ ##################
#~ ## Brute stacking
#~ ##################
#~ ## NMO
#~ Flow('nmo','sints vel',
     #~ '''
     #~ nmo velocity=${SOURCES[1]} half=n |
     #~ mutter v0=0.25 t0=0
     #~ ''')

#~ Plot('nmo',
       #~ '''
       #~ byte gainpanel=each | window j3=2 |
       #~ grey3 frame1=500 frame2=18 frame3=642 flat=n
       #~ title="NMOed Data"
       #~ label2=Offset label3=Midpoint
       #~ ''')
       
#~ ## Brute stacking
#~ Flow('bstack','nmo','stack')

#~ Plot('bstack',
       #~ '''
       #~ grey title="Brute stacking" labelfat=4 font=2 titlefat=4
       #~ ''')

#~ #################
#~ ## Migration
#~ #################

#~ Flow('vel2','vel',
     #~ 'window | dix rect1=20 rect2=2 weight=${SOURCES[0]}')

#~ ## Prestack Kirchhoff time migration
#~ Flow('tcmps','sints','transp memsize=1000 plane=23')
#~ Flow('pstm','tcmps vel2',
     #~ '''
     #~ mig2 vel=${SOURCES[1]} apt=50 antialias=1
     #~ ''',split=[3,10140,[0]],reduce='add')
#~ Plot('pstm',
       #~ '''
       #~ grey title="Prestack kirchhoff time migration"
       #~ labelfat=4 font=2 titlefat=4
       #~ ''')

#~ ###### post-stack

#~ Flow('post2','pstm vel2',
     #~ '''
     #~ stolt vel=${SOURCES[1]} extend=4
     #~ ''')

#~ Plot('post2',grey('Post-stack stolt'))

#~ ####### post-stack / brute ############

#~ Flow('post3','bstack vel2',
     #~ '''
     #~ stolt vel=${SOURCES[1]} extend=4
     #~ ''')

#~ Plot('post3',grey('Post-stack stolt'))

#~ #################
#~ ## Filter + Gain
#~ #################
#~ # 
#~ Flow('psgain','post3',
     #~ '''
     #~ pow pow1=2
     #~ ''')
     
#~ Plot('psgain',
       #~ '''
       #~ grey title="psgain"
       #~ labelfat=4 font=2 titlefat=4
       #~ ''')
