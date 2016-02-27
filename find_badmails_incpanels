#set -ex
# Works on cPanel/WHM server, searching thru 15 newest cpanel accounts for files with bad code in them & report bad files via email
# In same directry like this script, should be file "find_badware_incpanels_phrasses" which should contain all malicious phrasses that will be searched in new cpanels. All single and double quotation marks must be commented out by slash, example: /"something   . And be one bad phrasse per line. // EDIT, badlist is no longer used, we report all new mailfiles except ones containing whitephrasses
# To make this script working regularly, automatically, setup this script as cronjob.


# variables (usually only "adminmail" and "thispath" needs to be editted)
adminmail=YOUR@gmail.com
thispath=/root/badwarefinder/mailsfinder
searchbuffer=$thispath/searchbuffer
latestmails=$thispath/latestmails
oldmails=$thispath/oldmails
reportcurrent=$thispath/reportcurrent
badphrasses=$thispath/badphrasses
whitephrasses=$thispath/whitephrasses

> $latestmails
> $reportcurrent
> $searchbuffer

###################################

# add 50 log entries containing 15 newest cpanels into file
tail -n 50 /var/cpanel/accounting.log | grep CREATE > /tmp/lastcpanelslog
ls -A1 /var/cpanel/suspended > /tmp/suspendedcpanels
grep -v -F -f /tmp/suspendedcpanels /tmp/lastcpanelslog > /tmp/lastcpanels

# cpanel account loop
while read logline;do
# Add cpanel username and its domains into report for reference

# echo "logline: $logline"
# echo "logline useronly:"
cpusr=$(echo "$logline" | tail -c-9)
#echo $cpusr

existdir=$(bash -c '[ -d /usr/local/apache/domlogs/$cpusr/ ] && echo "exist"') 2> /dev/null
#echo "existdir: $existdir"

# echo "$cpusr exist check done"

#if [[ "$existdir" == "exist" ]];then
#echo "$cpusr, $(ls /usr/local/apache/domlogs/$cpusr 2> /dev/null | grep -v / 2> /dev/null)" >> $reportcurrent 2> /dev/null
#fi

## echo "$(echo "$logline" | tail -c-9), $(echo $domlogdirexist)"

# search bad phrasses in cpanel account files
#ls -A1 /home/$(echo "$logline" | tail -c-9)/mail | grep -v Sent | grep -v Trash | grep -v Junk | grep -v Drafts | grep -v cur | grep -v new | grep -v tmp | grep -v maildirsize >> $searchbuffer
# find /home/$(echo "$logline" | tail -c-9)/mail ! -path "*/tmp*" ! -path "*/backup*" ! -path "*/usr*"
# find latest (last 25 hours) email files sent out/received and put them into latestmails file
/bin/nice -n 19 find /home/$(echo "$logline" | tail -c-9)/mail \( -path "*Sent*" -o -path "*new*" \) -iname "*$(hostname)*" -cmin -1500 | head -n1 | sort -nr >> $latestmails
# /bin/nice -n 19 find /home/$(echo "$logline" | tail -c-9)/mail >> $searchbuffer
# /bin/nice -n 19 grep -sRil "$phrasse" /home/$(echo "$logline" | tail -c-9) >> $reportcurrent
# /bin/nice -n 19 grep -sRil "$phrasse" /home/$(echo "$logline" | tail -c-9)/public_html >> $reportcurrent

done < /tmp/lastcpanels

# filter out mailfiles that contains phrasses from whitephrasses file
# empty refined/buffer file before start this loop
> $thispath/latestmailsrefined
while read whitephrasse;do
while read mailfile;do
if [[ "$(grep "$whitephrasse" $mailfile |wc -l)" -gt "0" ]];then
# whitephrasse found in this mailfile, do not report this mailfile, remove it from latestmails file
cat $latestmails|grep -v "$mailfile" >> $thispath/latestmailsrefined
fi
done < $latestmails
done < $whitephrasses
# deduplicate latestmails file
sort -u $thispath/latestmailsrefined > $latestmails

# compare archive with current report and output only new file pathes
newfilesonly=$(/bin/nice -n 19 awk 'FNR==NR{a[$0]++;next}!a[$0]' $oldmails $latestmails)
# copy currently found file pathes into report archive (oldmails)
cat $latestmails | grep mail >> $oldmails

# report results via email
if [ "$(echo $newfilesonly | grep mail | wc -l)" -gt "0" ];then

while read mailfile;do
echo "This is new mailfile: $mailfile

$(head -n 80 $mailfile)

End of mailfile. Hope it is clean beatifull message. Contains hard to read HTML? use cmd /root/bulterierkathtm $mailfile

Base64? If yes, here it is:
$(sed -n '/base64/,/------/p' $mailfile | sed '/^$/d;s/ //g' | sed '1d' | head -n -1 | base64 --decode)" | mail -s "New mail to check" $adminmail
done < $latestmails
fi
# this is the end
