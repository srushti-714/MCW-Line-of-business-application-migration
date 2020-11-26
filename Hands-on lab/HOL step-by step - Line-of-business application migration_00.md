![Microsoft Cloud Workshops](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png "Microsoft Cloud Workshops")

<div class="MCWHeader1">
Line-of-business application migration
</div>

<div class="MCWHeader2">
Hands-on lab step-by-step
</div>

<div class="MCWHeader3">
November 2020
</div>


Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2020 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents** 

<!-- TOC -->

- [Line-of-business application migration hands-on lab step-by-step](#line-of-business-application-migration-hands-on-lab-step-by-step)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Overview](#overview)
  - [Solution architecture](#solution-architecture)
  - [Requirements](#requirements)
  - [Before the hands-on lab](#before-the-hands-on-lab)
  - [Exercise 1: Discover and assess the on-premises environment](#exercise-1-discover-and-assess-the-on-premises-environment)
    - [Task 1: Create the Azure Migrate project and add assessment and migration tools](#task-1-create-the-azure-migrate-project-and-add-assessment-and-migration-tools)
    - [Task 2: Deploy the Azure Migrate appliance](#task-2-deploy-the-azure-migrate-appliance)
    - [Task 3: Configure the Azure Migrate appliance](#task-3-configure-the-azure-migrate-appliance)
    - [Task 4: Create a migration assessment](#task-4-create-a-migration-assessment)
    - [Task 5: Configure dependency visualization](#task-5-configure-dependency-visualization)
    - [Task 6: Explore dependency visualization](#task-6-explore-dependency-visualization)
  - [Exercise 2: Migrate the Application Database](#exercise-2-migrate-the-application-database)
    - [Task 1: Register the Microsoft.DataMigration resource provider](#task-1-register-the-microsoftdatamigration-resource-provider)
    - [Task 2: Create an Azure SQL Database](#task-2-create-an-azure-sql-database)
    - [Task 3: Create the Database Migration Service](#task-3-create-the-database-migration-service)
    - [Task 4: Assess the on-premises database using Data Migration Assistant](#task-4-assess-the-on-premises-database-using-data-migration-assistant)
    - [Task 5: Create a DMS migration project](#task-5-create-a-dms-migration-project)
    - [Task 6: Migrate the database schema](#task-6-migrate-the-database-schema)
    - [Task 7: Migrate the on-premises data](#task-7-migrate-the-on-premises-data)
  - [Exercise 3: Migrate the application and web tiers using Azure Migrate: Server Migration](#exercise-3-migrate-the-application-and-web-tiers-using-azure-migrate-server-migration)
    - [Task 1: Create a Storage Account](#task-1-create-a-storage-account)
    - [Task 2: Create a Virtual Network](#task-2-create-a-virtual-network)
    - [Task 3: Register the Hyper-V Host with Azure Migrate: Server Migration](#task-3-register-the-hyper-v-host-with-azure-migrate-server-migration)
    - [Task 4: Enable Replication from Hyper-V to Azure Migrate](#task-4-enable-replication-from-hyper-v-to-azure-migrate)
    - [Task 5: Configure static internal IP addresses for each VM](#task-5-configure-static-internal-ip-addresses-for-each-vm)
    - [Task 6: Server migration](#task-6-server-migration)
    - [Task 7: Enable Azure Bastion](#task-7-enable-azure-bastion)
    - [Task 8: Configure the database connection](#task-8-configure-the-database-connection)
    - [Task 9: Configure the public IP address and test the SmartHotel application](#task-9-configure-the-public-ip-address-and-test-the-smarthotel-application)
    - [Task 10: Post-migration steps](#task-10-post-migration-steps)
  - [After the hands-on lab](#after-the-hands-on-lab)
    - [Task 1: Clean up resources](#task-1-clean-up-resources)

<!-- /TOC -->

# Line-of-business application migration hands-on lab step-by-step 

