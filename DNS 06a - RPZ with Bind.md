logging {
        channel null { 
		null; 
	};

    channel bindlog { 
		file "bind.log"; 
		print-time yes; 
		print-category yes; 
		print-severity yes; 
		severity info; 
	};

    channel rpzlog {
        file "rpz.log" versions unlimited size 1000m;
        print-time yes;
        print-category yes;
        print-severity yes;
        severity info;
    };

    category default { bindlog; };
    category general { bindlog; };
    category database { null; };
    category config { bindlog; };
    category resolver { null; };
    category xfer-in { bindlog; };
    category xfer-out { bindlog; };
    category notify { bindlog; };
    category client { null; };
    category unmatched { null; };
    category network { bindlog; };
    category update { bindlog; };
    category update-security { bindlog; };
    category queries { null; };
    category dispatch { null; };
    category lame-servers { null; };
    category delegation-only { bindlog; };
    category edns-disabled { null; };
	category rpz { rpzlog; };
};

options {
    directory "/var/cache/bind";
    dnssec-validation no;
    listen-on port 53 { localhost; 100.100.0.0/16; };
    listen-on-v6 port 53 { localhost; fd89:59e0::/32; };
    allow-query { any; };
    recursion yes;

    allow-transfer{ none; };
	ixfr-from-differences yes;
	empty-zones-enable yes;

	response-policy {
		zone "rpz.local";
	};
};

In named.conf.local

zone "rpz.local" {
	type master;
	file "db.rpz.local";
	allow-update { none; };
	allow-transfer { none; };
	allow-query { localhost; };
};

zone "rpz.te-labs.training" {
	type slave;
	file "db.rpz.te-labs.training";
	masters { 
		AWS;
		AWS;
	};
	allow-transfer { none; };
	allow-query { localhost; };
};

#------------------------------------------------------------------------------
# Root hints
#------------------------------------------------------------------------------

zone "." {
	type hint;
	file "root.cache";
};

