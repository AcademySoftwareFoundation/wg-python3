# Python 3 Working Group Request for Closure

## Current group purpose

See README.md.

## Assessment of deliverables

* Regular updates to the VES committee list of VFX packages and applications that support Python 3.
  This includes packages that are used frequently in VFX and do not already have Python 3 support.
    * Vendors present in the working group updated the spreadsheet when possible.
    * Some information in the spreadsheet remains outdated,
      for software from vendors not present in the working group.

* A document that collates best practices and advice to consider when transitioning to Python 3,
  not necessarily specific to Python 3.
    * Given that the Python 3 community at large has already created good transition guides,
      we collated a set of existing resources:
      https://github.com/AcademySoftwareFoundation/wg-python3/blob/master/deliverables/resources.md
    * However we also created a basic transition guide as a starting point
      for larger studios to base their own guide off of:
      https://github.com/AcademySoftwareFoundation/wg-python3/blob/master/deliverables/transition_guide.md
    * We found that documentation around transitioning usage of the C-API was lacking however,
      so SideFX documented their experience of porting their C code to Python 3:
      https://github.com/AcademySoftwareFoundation/wg-python3/blob/master/references/sidefx_cpython_notes.md

* A collection of common problems and their respective solutions found
  when transitioning to Python 3.
    * We created a document of solution to the few VFX specific problems
      that studios and DCC vendors came across in their transitions:
      https://github.com/AcademySoftwareFoundation/wg-python3/blob/master/deliverables/common_problems.md

* A regularly maintained list of risk factors to the VFX industry
  in its timely transition to Python 3.
    * A formal list was never created.
      However studios often noted the fact that they were not making significant progress
      in transitions because they were not a priority of the company.
      This will likely remain the case until more vendors drop Python 2 support.
      However some vendors are not willing to drop Python 2 support so long as customers want it.
    * Not all third party software is Python 3 compatible, including some DCCs.
    Some software that has announced future Python 3 support has not given a date
    for when Python 2 support will be dropped.

## Assessment of goals

* Share and develop best practices for transitioning to Python 3.
    * We believe that the deliverables mentioned in the previous section
      are enough to consider this goal as achieved.

* Coordinate transition efforts between software developers and vendors.
    * Studios gave regular status updates to each other about our progress of
      transitioning to Python 3.
      It may be useful to continue getting updates from studios to help DCC vendors
      prioritise (or deprioritise) their Python 2 support,
      and possibly to encourage studios that are falling behind to pick up the pace
      of their own transitions.
    * It is difficult for DCC vendors to give updates on Python 2 & 3 support for legal reasons.
* Articulate and share common experiences to avoid duplicate efforts.
    * We believe that the "common problems" document meets this goal,
      though we found few common problems specific to VFX.

## Loose ends

We had been in contact with The Foundry to try and get a statement on
the future of Python support similar to what Autodesk had produced
(https://github.com/AcademySoftwareFoundation/wg-python3/blob/master/deliverables/vendor_statements/autodesk.md).
The Foundry committed to making such a statement but they haven't released one yet
and the group hasn't followed up.

## Other notes

* Few larger studios are making significant progress with the Python 3 transition because,
  as mentioned previously, few studios are prioritising the work.
  WDAS and DNEG are the only two larger studios that had made any significant progress
  in the last status update.
  Other larger studios were making some small steps to making it happen.
  Most smaller studios are Python 3 ready.

* The fact that some software is still not Python 3 compatible is a significant blocker
in achieving full Python 3 support across VFX.
The fact that lots of software still supports Python 2 discourages studios
from wanting to transition to Python 3.
This seems unlikely to change even with the VFX platform having already dropped Python 2 support.
It is difficult for the working group alone to encourage all vendors and studios
to transition to Python 3.
